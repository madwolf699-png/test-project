# プロジェクトのKubernetes登録

このプロジェクト全体をはKubernetesに登録する手順について記載します。

## 0. Dockerイメージをビルドし、イメージファイルを作成
```bash
docker build -t [repository-name]-backend:latest -f docker/backend/Dockerfile.prod backend/
docker build -t [repository-name]-frontend:latest -f docker/frontend/Dockerfile.prod frontend/
docker save [repository-name]-backend:latest -o [repository-name]-backend.tar
docker save [repository-name]-frontend:latest -o [repository-name]-frontend.tar
```

## 1. containerに登録する場合(inuxn等)
- kubernetesと同じマシンに登録する場合
  - Namespace
    ```bash
    kubectl apply -f k8s/namespace.yaml
    ```
  - backend
    ```bash
    #containerdにイメージを入れる
    sudo ctr -n k8s.io images import [repository-name]-backend.tar
    sudo ctr -n k8s.io images ls | grep [repository-name]
    kubectl delete pod -n [repository-name] --all
    kubectl apply -f k8s/backend/

    #kubectl port-forward svc/backend 8080:80 -n [repository-name]
    #http://localhost:8080

    kubectl run curl-test \
    -n [repository-name] \
    --rm -it \
    --image=curlimages/curl -- sh

    curl -v http://backend:8080
    ```

  - frontend
    ```bash
    #containerdにイメージを入れる
    sudo ctr -n k8s.io images import [repository-name]-frontend.tar
    sudo ctr -n k8s.io images ls | grep [repository-name]
    kubectl delete pod -n [repository-name] --all
    kubectl apply -f k8s/frontend/

    #kubectl port-forward svc/backend 8080:80 -n [repository-name]
    http://localhost:30080

    kubectl exec -it -n [repository-name] deploy/frontend -- sh
    curl -v http://backend:8080
    http://localhost:/30080
     ```
## 2. Docker Desktopに登録する場合(Windows等)
- kubernetesと同じマシンに登録する場合
  - Namespace
    ```bash
    kubectl apply -f k8s/namespace.yaml
    ```
  - backend
    ```bash
    docker load -i [repository-name]-backend.tar
    docker images | grep [repository-name]-backend
    kubectl delete pod -n [repository-name] --all
    kubectl apply -f k8s/backend/

    kubectl run curl-test \
    -n [repository-name] \
    --rm -it \
    --image=curlimages/curl -- sh

    curl -v http://backend:8080
    ```

  - frontend
    ```bash
    docker load -i [repository-name]-frontend.tar
    docker images | grep [repository-name]-frontend
    kubectl delete pod -n [repository-name] --all
    kubectl apply -f k8s/frontend/

    kubectl exec -it -n [repository-name] deploy/frontend -- sh
    curl -v http://backend:8080
    http://localhost:/30080
     ```
