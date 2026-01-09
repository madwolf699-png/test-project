# プロジェクトのKubernetes登録

このプロジェクト全体をはKubernetesに登録する手順について記載します。

## 0. Dockerイメージをビルド
```bash
docker build -t [repository-name]-backend:latest -f docker/backend/Dockerfile.prod backend/
docker build -t [repository-name]-frontend:latest -f docker/frontend/Dockerfile.prod frontend/
```

## 1. kubernetesと同じマシンに登録する
- Namespace
  ```bash
  kubectl apply -f k8s/namespace.yaml
  ```
- backend
  ```bash
  #containerdにイメージを入れる
  docker save test-project-backend:latest -o backend.tar
  sudo ctr -n k8s.io images import backend.tar
  sudo ctr -n k8s.io images ls | grep test-project
  kubectl delete pod -n test-project --all
  kubectl apply -f k8s/backend/

  #kubectl port-forward svc/backend 8080:80 -n test-project
  #http://localhost:8080

  kubectl run curl-test \
  -n test-project \
  --rm -it \
  --image=curlimages/curl -- sh

  curl -v http://backend:8080
  ```

- frontend
  ```bash
  #containerdにイメージを入れる
  docker save test-project-frontend:latest -o frontend.tar
  sudo ctr -n k8s.io images import frontend.tar
  sudo ctr -n k8s.io images ls | grep test-project
  kubectl delete pod -n test-project --all
  kubectl apply -f k8s/frontend/

  #kubectl port-forward svc/backend 8080:80 -n test-project
  http://localhost:30080

  kubectl exec -it -n test-project deploy/frontend -- sh
  curl -v http://backend:8080
  ```

- アクセス確認
  ```bash
  kubectl get pods -n test-project
  kubectl get svc -n test-project
  http://<RaspberryPiのIP>:30080
  ```
