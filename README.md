# Fibonacci Calculator - Orchestrated

A multi-tier Fibonacci calculator demonstrating advanced Kubernetes orchestration with microservices architecture. Built with React, Express.js, Redis, and PostgreSQL, containerized with Docker and deployed on Google Kubernetes Engine (GKE) with automated CI/CD.

## 🏗️ Architecture

This application demonstrates a modern orchestrated microservices architecture:

- **Frontend**: React.js single-page application
- **Backend API**: Express.js REST API server  
- **Worker Service**: Background Fibonacci calculator
- **Cache**: Redis for storing calculation results
- **Database**: PostgreSQL for persistent storage
- **Orchestration**: Kubernetes with services and ingress
- **Deployment**: Google Kubernetes Engine (GKE)
- **CI/CD**: GitHub Actions

### System Diagram

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   React Client  │────│  Express Server  │────│ Background      │
│   (Frontend)    │    │     (API)        │    │ Worker          │
│   Port: 3000    │    │   Port: 5000     │    │ (Fibonacci)     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                        │
         │                       │                        │
         └───────────────────────┼────────────────────────┘
                                 │
                    ┌────────────┴─────────────┐
                    │                          │
            ┌───────▼──────┐          ┌────────▼─────┐
            │    Redis     │          │ PostgreSQL   │
            │   (Cache)    │          │ (Database)   │
            │   Port: 6379 │          │  Port: 5432  │
            └──────────────┘          └──────────────┘
```

### Components

1. **Client (React + Nginx)**: Frontend SPA served by Nginx
2. **Server (Express.js)**: REST API handling HTTP requests
3. **Worker (Node.js)**: Background processor for Fibonacci calculations
4. **Redis**: Caching layer for computed Fibonacci values
5. **PostgreSQL**: Persistent storage for request history

## 🚀 Features

- Calculate Fibonacci numbers for any index (up to 40)
- Real-time display of previously calculated values
- Responsive React frontend with routing
- Background processing for intensive calculations
- Persistent storage of calculation history
- Scalable microservices architecture
- Kubernetes-native deployment with auto-scaling
- SSL termination and load balancing

## 📋 Tech Stack

| Category | Technologies |
|----------|-------------|
| Frontend | React 18, React Router, Axios |
| Backend | Node.js, Express.js, CORS |
| Worker | Node.js background service |
| Database | PostgreSQL |
| Cache | Redis |
| Orchestration | Kubernetes, Docker |
| Cloud Platform | Google Kubernetes Engine (GKE) |
| SSL/Ingress | nginx-ingress, cert-manager, Let's Encrypt |
| CI/CD | GitHub Actions |

## 🛠️ Local Development Setup

### Prerequisites

- Minikube installed and running
- kubectl configured  
- Docker for building images

### Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd fibonacci-calculator-orchestrated
   ```

2. **Start Minikube and configure Docker**
   ```bash
   minikube start
   eval $(minikube docker-env)
   ```

3. **Create Kubernetes secret**
   ```bash
   kubectl create secret generic pgpassword --from-literal PGPASSWORD=postgres_password
   ```

4. **Build Docker images locally**
   ```bash
   docker build -t fibonacci-calculator-client:latest -f ./client/Dockerfile ./client
   docker build -t fibonacci-calculator-server:latest -f ./server/Dockerfile ./server
   docker build -t fibonacci-calculator-worker:latest -f ./worker/Dockerfile ./worker
   ```

5. **Deploy to Minikube**
   ```bash
   kubectl apply -f k8s-local/
   ```

6. **Access the application**
   ```bash
   # Get Minikube IP
   minikube ip
   # Open http://<minikube-ip> in your browser
   ```

### Development Workflow

The local development setup includes:

- **Live rebuilds**: Rebuild Docker images when code changes
- **Kubernetes services**: All inter-service communication via K8s services
- **Persistent volumes**: Database data persists across pod restarts
- **Local image builds**: No external registry required for development

### Stopping the Environment

```bash
# Delete all local deployments
kubectl delete -f k8s-local/

# Stop Minikube
minikube stop
```

> **Note**: The `k8s-local` directory contains configurations optimized for local Minikube development, excluding SSL certificates and ingress that require external domains.

## 🔧 Application Flow

1. **User Input**: User enters a number in the React frontend
2. **API Request**: Frontend sends POST request to `/api/values`
3. **Data Storage**:
   - Number stored in PostgreSQL database
   - Placeholder stored in Redis cache
4. **Background Processing**: Worker service calculates Fibonacci number
5. **Result Storage**: Calculated value stored in Redis
6. **Display**: Frontend fetches and displays results from both data stores

## 🚀 CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/deploy.yaml`) automates:

### Build Phase

1. **Test Execution**: Runs React test suite
2. **Image Building**: Creates Docker images for all services:
   - `client` (React production build)
   - `server` (Express API)
   - `worker` (Fibonacci calculator)

### Deployment Phase

3. **Image Registry**: Pushes images to Docker Hub
4. **GKE Authentication**: Connects to Google Kubernetes Engine
5. **Rolling Deployment**: Updates Kubernetes deployments with zero downtime

### Triggered By

- Push to `main` branch
- Automatic deployment on successful builds

## 🚀 How GKE Deploys the Application

Google Kubernetes Engine orchestrates the application using Kubernetes manifests:

1. **Deployment Package**: GitHub Actions applies Kubernetes manifests from `k8s/` directory
2. **Container Orchestration**: GKE creates pods from Docker Hub images:
   - `nginx-ingress` (external traffic routing)
   - `client` - React frontend served by nginx
   - `server` - Express API connected to PostgreSQL/Redis services
   - `worker` - Background Fibonacci calculator
3. **Service Integration**: Kubernetes services provide internal DNS and load balancing
4. **SSL & Ingress**: cert-manager automatically provisions Let's Encrypt certificates

The Kubernetes manifests serve as the deployment blueprint, defining desired state for all application components.

## ☁️ Google Cloud Infrastructure Setup

### Required GCP Services

1. **Google Kubernetes Engine (GKE)** - Managed Kubernetes cluster
2. **Cloud SQL PostgreSQL** - Managed database (optional, or use in-cluster)
3. **Memorystore Redis** - Managed Redis cache (optional, or use in-cluster)
4. **Cloud DNS** - Domain management for SSL certificates
5. **Service Account** - For GitHub Actions deployment

### Detailed Setup Instructions

#### 1. GKE Cluster Setup

1. Navigate to GKE console in Google Cloud
2. Click "Create Cluster"
3. Choose "GKE Standard" for full Kubernetes features
4. Configuration:
   - **Cluster Name**: `fibonacci-calculator-cluster`
   - **Location Type**: Regional (for high availability)
   - **Region**: `us-central1` (or your preferred region)
   - **Node Pool**: 
     - Machine type: `e2-medium` (2 vCPU, 4GB RAM)
     - Number of nodes: 3
   - **Networking**: Enable HTTP load balancing
   - **Security**: Enable Workload Identity

#### 2. Install Required Controllers

1. **Install nginx-ingress controller**:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
   ```

2. **Install cert-manager for SSL**:
   ```bash
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
   ```

#### 3. Domain Setup

1. **Purchase domain** or use existing domain
2. **Configure Cloud DNS**:
   - Create DNS zone for your domain
   - Point domain nameservers to Google Cloud DNS
3. **Get ingress IP**:
   ```bash
   kubectl get service -n ingress-nginx ingress-nginx-controller
   ```
4. **Create A record**: Point your domain to the ingress external IP

#### 4. Environment Variables

Configure the following in your Kubernetes secrets and deployments:

| Variable | Value | Description |
|----------|-------|-------------|
| REDIS_HOST | redis-cluster-ip-service | Redis service hostname |
| REDIS_PORT | 6379 | Redis port |
| PGUSER | postgres | Database username |
| PGPASSWORD | (from secret) | Database password |
| PGHOST | postgres-cluster-ip-service | PostgreSQL service hostname |
| PGDATABASE | postgres | Database name |
| PGPORT | 5432 | PostgreSQL port |

#### 5. Service Account for GitHub Actions

1. **Create service account**:
   ```bash
   gcloud iam service-accounts create fibonacci-calculator-deployer \
     --description="Service account for GitHub Actions deployment" \
     --display-name="Fibonacci Calculator Deployer"
   ```

2. **Grant permissions**:
   ```bash
   gcloud projects add-iam-policy-binding PROJECT_ID \
     --member="serviceAccount:fibonacci-calculator-deployer@PROJECT_ID.iam.gserviceaccount.com" \
     --role="roles/container.developer"
   ```

3. **Create and download key**:
   ```bash
   gcloud iam service-accounts keys create service-account.json \
     --iam-account=fibonacci-calculator-deployer@PROJECT_ID.iam.gserviceaccount.com
   ```

#### 6. GitHub Repository Secrets

Required secrets for CI/CD pipeline:

```
GKE_SA_KEY=<base64-encoded-service-account-json>
DOCKER_USERNAME=<your-dockerhub-username>  
DOCKER_PASSWORD=<your-dockerhub-password>
```

### Production vs Local Configuration

**Production Additional Components (`k8s/`):**
- `certificate.yaml` - SSL certificate configuration
- `ingress-service.yaml` - External traffic routing with SSL
- `issuer.yaml` - Let's Encrypt certificate issuer
- Docker Hub images with `imagePullPolicy: Always`

**Local Configuration (`k8s-local/`):**
- Local Docker images with `imagePullPolicy: Never`  
- No SSL/TLS configuration
- No external ingress

## 📊 Monitoring and Troubleshooting

### Health Checks

- **GKE Dashboard**: Monitor cluster health in Google Cloud Console
- **Application Logs**: View pod logs with `kubectl logs <pod-name>`
- **Service Status**: Check service endpoints with `kubectl get services`
- **Ingress Status**: Monitor ingress with `kubectl get ingress`

### Common Issues

1. **502 Bad Gateway**: Check service configurations and pod readiness
2. **Database Connection**: Verify secrets and service DNS resolution  
3. **SSL Certificate**: Ensure domain DNS is correctly configured
4. **ImagePullBackOff**: Verify Docker Hub credentials and image names

## 🔄 Deployment Process

1. **Make Changes**: Modify code in any service
2. **Commit & Push**:
   ```bash
   git add .
   git commit -m "your changes"
   git push origin main
   ```
3. **Monitor**: Check GitHub Actions for build status
4. **Verify**: Visit your domain when deployment completes

## 📁 Project Structure

```
├── client/                 # React frontend
│   ├── src/
│   ├── public/
│   ├── Dockerfile
│   └── Dockerfile.dev
├── server/                 # Express API
│   ├── index.js
│   ├── keys.js
│   ├── Dockerfile
│   └── Dockerfile.dev
├── worker/                 # Fibonacci calculator
│   ├── index.js
│   ├── keys.js
│   ├── Dockerfile
│   └── Dockerfile.dev
├── k8s/                    # Production Kubernetes manifests
│   ├── client-deployment.yaml
│   ├── server-deployment.yaml
│   ├── worker-deployment.yaml
│   ├── postgres-deployment.yaml
│   ├── redis-deployment.yaml
│   ├── *-cluster-ip-service.yaml
│   ├── ingress-service.yaml
│   ├── certificate.yaml
│   └── issuer.yaml
├── k8s-local/              # Local development manifests
│   └── (same as k8s/ minus SSL/ingress)
└── .github/workflows/
    └── deploy.yaml         # CI/CD pipeline
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with Minikube
5. Submit a pull request

## 📄 License

This project is part of a learning exercise demonstrating Kubernetes orchestration, microservices architecture, and cloud deployment patterns.
