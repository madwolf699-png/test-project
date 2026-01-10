# プロジェクトのKubernetes登録

このプロジェクト全体をはKubernetesに登録する手順について記載します。

## 0. 全体構成
Kubernetes (Raspberry Pi 4)
 ├─ prometheus (metrics収集)
 ├─ alertmanager
 ├─ grafana (可視化)
 ├─ node-exporter (ノード監視)
 └─ kube-state-metrics (Pod / Service 監視)

- 前提条件
  - Raspberry Pi 4 + Ubuntu
  - Kubernetesクラスタ稼働
  - Pod/Serviceを2つデプロイ済
  - kubectlが使える
- 必要なもの
  - Helm v3
  - ARM64対応（kube-prometheusstackは対応済）

## 1. Helmのインストール
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## 2. Prometheus/Grafana用Namespace作成
```bash
kubectl create namespace monitoring
```

## 3. Helmリポジトリ追加
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## 4. kube-prometheus-stackをインストール
```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring
```

## 5.Pod起動確認
```bash
kubectl get pods -n monitoring
```

## 6.Grafanaへアクセスする（NodePort）
  - Grafana Serviceを確認
  ```bash
  kubectl get svc -n monitoring
  ```
  - NodePortに変更（外部アクセス用）
  ```bash
  kubectl edit svc monitoring-grafana -n monitoring

  spec:
    type: NodePort
  ```
  - ブラウザでアクセス
  ```bash
  http://<RaspberryPiのIP>:32000
  ```

## 7. Grafanaログイン情報
- 管理者パスワード取得
```bash
kubectl get secret monitoring-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
```

## 8. すでにダッシュボードは自動で入っている
- Kubernets Cluster Overview
- Node CPU/Memory/Disk
- Pod/Namespace使用量
- Service/Deployment状態

## 9. 自分のPod/Serviceを監視する
- Serviceにannotationsを追加
```bash
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
```

## 10. Prometheusでメトリクス確認
```bash
kubectl port-forward svc/monitoring-kube-prometheus-stack-prometheus \
  -n monitoring 9090:9090

http://localhost:9090
```
## A1. Raspberry Pi 4 特有の注意点
- メモリ対策
  - Pi 4（$GB以下）の場合
  ```bash
  kubectl edit deployment monitoring-kube-prometheus-stack-grafana -n monitoring

  resources:
  limits:
    memory: "512Mi"
  requests:
    memory: "256Mi"
  ```

## A2. 削除したい場合
```bash
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```

## A3. Raspberry Pi 4向け
- 一度削除
```bash
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```
- 軽量values.yamlを作成
```bash
grafana:
  replicas: 1
  resources:
    requests:
      cpu: 100m
      memory: 200Mi
    limits:
      memory: 300Mi

prometheus:
  prometheusSpec:
    replicas: 1
    resources:
      requests:
        cpu: 200m
        memory: 500Mi
      limits:
        memory: 800Mi
    retention: 6h

alertmanager:
  enabled: false

kubeStateMetrics:
  resources:
    requests:
      memory: 100Mi
    limits:
      memory: 200Mi

nodeExporter:
  resources:
    requests:
      memory: 50Mi
    limits:
      memory: 100Mi
```

- 軽量構成で再インストール
```bash
kubectl create namespace monitoring

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f prometheus-values.yaml
```
