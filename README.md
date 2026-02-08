# ChuanLiuBuXi

## 方式一：不在本地构建（推荐）

由 GitHub Actions 在每次推送到 `main` 时自动构建并推送镜像到 GitHub Container Registry (ghcr.io)，Argo CD 只负责从 Git 同步清单并部署。

**你需要做的：**

1. **改 Deployment 里的镜像地址**  
   编辑 `k8s/deployment.yaml`，把 `ghcr.io/OWNER/ChuanLiuBuXi:latest` 里的 `OWNER` 改成你的 **GitHub 用户名或组织名**（和仓库所属一致）。

2. **推送代码**  
   提交并 push 到 `main`，触发 Actions 构建、推送镜像。

3. **集群拉取 ghcr.io 镜像**  
   - 若为**公开镜像**：很多集群可直接拉取。  
   - 若为**私有镜像**：在 K8s 里创建 `imagePullSecrets`（用 GitHub PAT 或 GITHUB_TOKEN），并在 `k8s/deployment.yaml` 的 `spec.template.spec` 下加上 `imagePullSecrets`。

4. **在 Argo CD 里创建应用**  
   同下方「在 Argo CD 里创建应用」，Argo 会同步 `k8s/` 并部署，从 ghcr.io 拉取镜像。

之后只需 **git push**，CI 构建镜像 → Argo CD 同步 → 自动部署，无需本地构建。

---

## 方式二：在本地构建后运行

### 1. 构建并让集群能用镜像

**Docker Desktop / 本地 K8s：**
```bash
docker build -t chuanliubuxi:latest .
# 若 K8s 用本机 Docker：无需 push，保持 image 为 chuanliubuxi:latest 即可
```

**Kind：**
```bash
docker build -t chuanliubuxi:latest .
kind load docker-image chuanliubuxi:latest --name kind
```

**使用远程镜像仓库（Argo 部署的集群不在本机时）：**
```bash
docker build -t your-registry/chuanliubuxi:latest .
docker push your-registry/chuanliubuxi:latest
```
然后把 `k8s/deployment.yaml` 里的 `image` 改成 `your-registry/chuanliubuxi:latest`，并提交到 Git。

### 2. 推送代码到 Git

确保当前代码已提交并推送到你的 Git 仓库（Argo CD 会从该仓库拉取 `k8s/` 清单）。

### 3. 在 Argo CD 里创建应用

**方式 A：kubectl 应用 Application 清单**

编辑 `argocd/application.yaml`，把 `repoURL` 改成你的仓库地址，然后：
```bash
kubectl apply -f argocd/application.yaml
```

**方式 B：在 Argo CD UI 里新建**

- **Application name**: chuanliubuxi  
- **Project**: default  
- **Sync policy**: 可选 Automatic  
- **Repository URL**: 你的 Git 仓库地址  
- **Revision**: main（或你的默认分支）  
- **Path**: k8s  
- **Cluster**: in-cluster  
- **Namespace**: default  

保存后 Argo CD 会同步并部署。

### 4. 访问服务

集群内访问：`http://chuanliubuxi.default.svc.cluster.local/ping`  

本地端口转发后访问：
```bash
kubectl port-forward svc/chuanliubuxi 8080:80
# 浏览器打开 http://localhost:8080/ping
```