# Docker

```sql
Developer (Git Push Code)
          |
          v
      Git Server (GitHub / GitLab / Bitbucket)
          |
          | 1. Trigger Pipeline (CI)
          v
        CI Server (GitLab CI / Jenkins / GitHub Actions)
          |
          | 2. Build Docker Image
          | 3. Tag Image (version, commit, branch)
          |
          v
     Docker Registry (Harbor / GitLab Registry / ECR)
          |
          | 4. Push Image to Harbor
          v
         Harbor (Image Registry & Security Scan)
          |
          | 5. Pull Image by K8s during deploy
          v
      Kubernetes Cluster
          |
          | 6. Update Deployment → Rolling Update
          v
   Application Running in Pods
```

3 job chính của Harbor:

* Nhận image push từ CI
* Lưu trữ và version hóa image
* Cho K8s pull image khi chạy pod

K8s pod mới chạy

* Pod mới pull image: harbor/.../myapp:v2.5.1
* Healthcheck OK
* Pod cũ terminated

```sql

[1] Client 
      ↓
[2] External Load Balancer 
      ↓
[3] Kong Ingress Controller 
      ↓
[4] Kong Gateway (Proxy) 
      ↓
[5] Kubernetes Service (ClusterIP)
      ↓
[6] Pod (Containers inside)

```

Client gọi API
-> LB nhận request và điều phối đến các node K8s đang expose Kong
-> Kong Gateway: jwt (validate token), log (ES)
-> k8S: Lấy danh sách Pod có label tương ứng -> run