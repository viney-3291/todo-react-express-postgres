# Todo App — Production Deployment on AWS EKS

A fullstack Todo application deployed to production using a complete DevOps workflow: Docker, AWS ECR, AWS EKS (Kubernetes), and CI/CD with GitHub Actions.

This repo demonstrates taking an existing application codebase and deploying it to production-grade cloud infrastructure — the real workflow a DevOps/Platform engineer follows when handed a developer's code.

---

## 🏗️ Architecture

```
                         ┌─────────────────────┐
                         │   GitHub Actions     │
                         │   CI/CD Pipeline      │
                         └──────────┬───────────┘
                                    │ build & push
                                    ▼
                         ┌─────────────────────┐
                         │     AWS ECR          │
                         │  (Container Registry)│
                         └──────────┬───────────┘
                                    │ pull images
                                    ▼
   ┌────────────────────────────────────────────────────────┐
   │                  AWS EKS Cluster                        │
   │  ┌───────────────┐   ┌───────────────┐  ┌─────────────┐│
   │  │  Frontend Pod  │   │  Backend Pod   │  │ Postgres Pod││
   │  │  (React+Nginx) │←→ │ (Node/Express) │←→│             ││
   │  │   x2 replicas  │   │   x2 replicas  │  │  x1 replica ││
   │  └───────────────┘   └───────────────┘  └─────────────┘│
   └────────────────────────────┬─────────────────────────────┘
                                 │
                       AWS Load Balancer
                                 │
                            🌍 Internet
```

---

## 🧰 Tech Stack

**Application**
| Layer | Technology |
|---|---|
| Frontend | React, TypeScript, Tailwind CSS |
| Backend | Node.js, Express, TypeScript |
| Database | PostgreSQL |
| ORM | Prisma |

**Infrastructure & DevOps**
| Tool | Purpose |
|---|---|
| Docker | Multi-stage container builds |
| Docker Compose | Local multi-container orchestration |
| AWS ECR | Private container image registry |
| AWS EKS | Managed Kubernetes cluster |
| eksctl | EKS cluster provisioning |
| kubectl | Kubernetes resource management |
| GitHub Actions | CI/CD automation |

---

## 📦 What Was Built

- **Multi-stage Dockerfiles** for both frontend and backend — smaller image sizes, non-root users, separated build/runtime layers
- **Docker Compose** setup to run the full stack (frontend, backend, PostgreSQL) locally with a single command
- **AWS EKS cluster** provisioned with managed node groups across multiple availability zones
- **Kubernetes manifests** — Deployments with resource requests/limits and 2 replicas for backend/frontend, Services (`ClusterIP` for internal traffic, `LoadBalancer` for public access)
- **CI/CD pipeline** (GitHub Actions) that automatically builds Docker images, pushes to ECR, and performs a rolling deployment to EKS on every push to `main`

---

## 🚀 Running Locally

```bash
git clone https://github.com/viney-3291/todo-react-express-postgres.git
cd todo-react-express-postgres
docker-compose up --build
```

App available at **http://localhost:80**

---

## ☁️ Deploying to AWS EKS

```bash
# 1. Create the EKS cluster
eksctl create cluster --name todo-cluster --region us-east-1 \
  --nodegroup-name todo-nodes --node-type t3.medium \
  --nodes 2 --nodes-min 1 --nodes-max 3 --managed

# 2. Push images to ECR
docker build --platform linux/amd64 -t todo-backend ./backend
docker tag todo-backend:latest <ECR_URI>/todo-backend:latest
docker push <ECR_URI>/todo-backend:latest
# (repeat for frontend)

# 3. Deploy to the cluster
kubectl apply -f k8s/

# 4. Get the public URL
kubectl get services
```

---

## 🐛 Real Issues Resolved

| Issue | Fix |
|---|---|
| `ImagePullBackOff` on EKS | Attached `AmazonEC2ContainerRegistryReadOnly` IAM policy to the node role |
| `no match for platform in manifest` | Built images with `--platform linux/amd64` (Mac ARM64 → EKS AMD64) |
| Prisma `libssl` error in container | Added `RUN apk add --no-cache openssl` to the backend Dockerfile |
| Prisma engine not found for Alpine | Added `linux-musl-openssl-3.0.x` to `binaryTargets` in `schema.prisma` |

---

## 📂 Repository Structure

```
.
├── backend/                 # Express + Prisma API
│   └── Dockerfile
├── frontend/                # React + Vite client
│   └── Dockerfile
├── k8s/                     # Kubernetes manifests
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── postgres-deployment.yaml
│   └── postgres-service.yaml
├── .github/workflows/
│   └── deploy.yml            # CI/CD pipeline
└── docker-compose.yml         # Local development stack
```

---

## 📝 License

This project is based on the open-source [todo-react-express-postgres](https://github.com/nasso/todo-react-express-postgres) demo app, extended with a full production deployment pipeline to AWS.
