# Enterprise Monitoring & Auto Scaling AWS Project

## Technologies Used
- AWS EC2
- Docker
- Prometheus
- Grafana
- CloudWatch
- Load Balancer
- Auto Scaling
- Linux
- Flask

## Features
- Dockerized Flask application
- Monitoring with Prometheus & Grafana
- CloudWatch alerts
- Auto Scaling Group
- Application Load Balancer
- Centralized logging
- Infrastructure monitoring
- Production-style deployment

## Architecture
<img width="1536" height="1024" alt="aws-monitoring-sre-project-architecture-diagram" src="https://github.com/user-attachments/assets/88662e82-cb71-4398-9195-8366c385ff68" />


## Screenshots
<img width="1366" height="768" alt="grafana-all-metrics" src="https://github.com/user-attachments/assets/25dc14f7-941e-44ad-b61b-f821e93deae1" />
<img width="1366" height="768" alt="grafana-all-metrics" src="https://github.com/user-attachments/assets/88852d26-ad45-44b9-a26f-9b8990de466b" />
<img width="1366" height="768" alt="network-traffic" src="https://github.com/user-attachments/assets/e84bb598-ce36-4d7f-bfd5-8dd4fe5be8af" />


## Setup Instructions
# Deployment Instructions

## Step 1 — Launch EC2 Instance

- Launch Ubuntu EC2 instance in AWS
- Instance type: t3.small
- Configure Security Group:

Open ports:
- 22 (SSH)
- 80 (HTTP)
- 3000 (Grafana)
- 5000 (Application)
- 9090 (Prometheus)
- 9100 (Node Exporter)

---

## Step 2 — Connect to EC2
##bash
ssh -i newkey.pem ubuntu@<EC2-PUBLIC-IP>

## Install Docker

sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
newgrp docker
## verify
docker --version

## clone GitHub Repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY

##Build Flask Application
docker build -t production-app .
##run container
docker run -d \
--name production-app \
-p 5000:3000 \
production-app
##verify
curl localhost:5000

##Create Docker Network
docker network create monitoring
##run node exporter
docker run -d \
--name node-exporter \
--network monitoring \
-p 9100:9100 \
prom/node-exporter

##Configure Prometheus
mkdir prometheus-config
cd prometheus-config
nano prometheus.yml
##yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

##Run Prometheus
docker run -d \
--name prometheus \
--network monitoring \
-p 9090:9090 \
-v ~/prometheus-config/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus

##Run Grafana
docker run -d \
--name grafana \
--network monitoring \
-p 3000:3000 \
grafana/grafana
##access Grafana
http://<EC2-PUBLIC-IP>:3000


##Setup CloudWatch Agent
cd /tmp
wget https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
##create config
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json

##Configure GitHub Actions CI/CD
.github/workflows/deploy.yml

