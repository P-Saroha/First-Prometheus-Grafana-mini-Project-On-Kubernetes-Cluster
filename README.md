# Flask Metrics Monitoring with Prometheus & Grafana on Kubernetes

A comprehensive monitoring solution that demonstrates how to set up application metrics collection, monitoring, and visualization using Flask, Prometheus, and Grafana on a Kubernetes cluster (Minikube).

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Detailed Setup](#detailed-setup)
- [Accessing Services](#accessing-services)
- [Monitoring Metrics](#monitoring-metrics)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🎯 Overview

This project showcases a complete monitoring pipeline that includes:

- **Flask Application**: A simple web application that exposes custom metrics
- **Prometheus**: Time-series database for collecting and storing metrics
- **Grafana**: Visualization dashboard for monitoring and alerting
- **Kubernetes**: Container orchestration platform (using Minikube for local development)

The Flask application generates simulated metrics including:
- Total API requests counter
- Request processing latency
- Model prediction success rate

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│                 │    │                 │    │                 │
│   Flask App     │───▶│   Prometheus    │───▶│    Grafana      │
│  (Port: 5000)   │    │  (Port: 9090)   │    │  (Port: 3000)   │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │                 │
                    │   Kubernetes    │
                    │   (Minikube)    │
                    │                 │
                    └─────────────────┘
```

## 🛠️ Prerequisites

Before running this project, ensure you have:

- **Docker Desktop**: For containerization
- **Minikube**: Local Kubernetes cluster
- **kubectl**: Kubernetes command-line tool
- **Helm**: Package manager for Kubernetes
- **Python 3.9+**: For local development (optional)
- **PowerShell**: For Windows users

### Installation Commands

```bash
# Install Docker Desktop (download from official website)

# Install Minikube
winget install Kubernetes.minikube

# Install kubectl
winget install Kubernetes.kubectl

# Install Helm
winget install Helm.Helm
```

## 📁 Project Structure

```
First-Prometheus-Grafana-mini-Project-On-Kubernetes-Cluster/
├── app.py                 # Flask application with metrics endpoint
├── dockerfile            # Docker configuration for Flask app
├── flask-app.yaml        # Kubernetes deployment and service manifests
├── projectflow.txt       # Step-by-step project execution guide
├── README.md            # This file
├── LICENSE              # Project license
└── myenv/               # Python virtual environment
```

## 🚀 Quick Start

1. **Clone the repository**:
   ```bash
   git clone https://github.com/P-Saroha/First-Prometheus-Grafana-mini-Project-On-Kubernetes-Cluster.git
   cd First-Prometheus-Grafana-mini-Project-On-Kubernetes-Cluster
   ```

2. **Start Minikube**:
   ```bash
   minikube start
   ```

3. **Build and deploy Flask app**:
   ```bash
   docker build -t flask-metrics-app:latest .
   minikube image load flask-metrics-app:latest
   kubectl apply -f flask-app.yaml
   ```

4. **Install Prometheus**:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
   ```

5. **Install Grafana**:
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm install grafana grafana/grafana -n monitoring
   ```

## 📖 Detailed Setup

### Step 1: Flask Application Setup

The Flask application (`app.py`) includes:
- A metrics endpoint at `/metrics` that returns Prometheus-formatted metrics
- Simulated business metrics (requests, latency, success rate)
- Health check endpoints

### Step 2: Docker Containerization

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

### Step 3: Kubernetes Deployment

The `flask-app.yaml` contains:
- Deployment configuration with resource limits
- NodePort service for external access
- Proper labels for Prometheus service discovery

### Step 4: Prometheus Configuration

Prometheus is configured to scrape metrics from the Flask application:
- Service discovery via Kubernetes API
- Custom scrape configurations
- Data retention policies

### Step 5: Grafana Dashboard Setup

- Pre-configured dashboards for Flask metrics
- Prometheus as a data source
- Custom panels and visualizations

## 🌐 Accessing Services

### Flask Application
```bash
# Get the Flask app URL
minikube service flask-metrics-app --url
# Access metrics at: http://<URL>/metrics
```

### Prometheus Dashboard
```bash
# Port forward Prometheus
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
# Access at: http://localhost:9090
```

### Grafana Dashboard
```bash
# Port forward Grafana
kubectl port-forward svc/grafana -n monitoring 3000:80
# Access at: http://localhost:3000
# Username: admin
# Get password:
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

## 📊 Monitoring Metrics

The application exposes the following metrics:

### Custom Application Metrics
- `total_api_requests_total`: Counter of total API requests
- `request_processing_latency_seconds`: Gauge for request processing time
- `model_prediction_success_rate`: Gauge for ML model success rate

### System Metrics (via Prometheus Node Exporter)
- CPU usage
- Memory usage
- Disk I/O
- Network statistics

### Kubernetes Metrics (via kube-state-metrics)
- Pod status and resource usage
- Deployment health
- Service availability

## 🔧 Configuration

### Prometheus Scrape Configuration

To manually configure Prometheus to scrape your Flask app:

1. Edit the Prometheus ConfigMap:
   ```bash
   kubectl edit configmap prometheus-server -n monitoring
   ```

2. Add the scrape configuration:
   ```yaml
   scrape_configs:
     - job_name: 'flask-app'
       static_configs:
         - targets: ['<FLASK_SERVICE_IP>:5000']
   ```

3. Restart Prometheus:
   ```bash
   kubectl rollout restart deployment prometheus-server -n monitoring
   ```

### Grafana Data Source

1. Login to Grafana
2. Go to Configuration > Data Sources
3. Add Prometheus data source
4. URL: `http://prometheus-server.monitoring.svc.cluster.local:80`
5. Save & Test

## 🐛 Troubleshooting

### Common Issues

**Docker not running**:
```
Error: error during connect: Head "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/_ping"
```
*Solution*: Start Docker Desktop

**Helm not found**:
```
helm : The term 'helm' is not recognized
```
*Solution*: Restart PowerShell or use full path to helm.exe

**Pods not starting**:
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

**Port forwarding issues**:
- Ensure pods are running: `kubectl get pods -n monitoring`
- Check if ports are already in use: `netstat -an | findstr :9090`

### Useful Commands

```bash
# Check all resources
kubectl get all -n monitoring

# View logs
kubectl logs -f deployment/flask-metrics-app
kubectl logs -f deployment/prometheus-server -n monitoring

# Debug services
kubectl describe svc flask-metrics-app
kubectl describe svc prometheus-server -n monitoring

# Clean up
kubectl delete -f flask-app.yaml
helm uninstall prometheus -n monitoring
helm uninstall grafana -n monitoring
minikube stop
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📚 Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [Flask Documentation](https://flask.palletsprojects.com/)

## 🎯 Learning Objectives

This project helps you understand:
- Container orchestration with Kubernetes
- Application monitoring and observability
- Time-series data collection and storage
- Dashboard creation and visualization
- DevOps best practices for monitoring

---

**Happy Monitoring!** 🚀📊
