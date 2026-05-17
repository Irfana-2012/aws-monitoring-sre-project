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
<img width="1366" height="667" alt="grafana-all-metrics" src="https://github.com/user-attachments/assets/adbf9510-0a63-454d-9898-188023b554bd" />
<img width="1366" height="697" alt="import-node-exporter" src="https://github.com/user-attachments/assets/8df0b816-8d62-4fc3-a219-d11822d668ff" />
<img width="1366" height="672" alt="network-traffic" src="https://github.com/user-attachments/assets/6c4de304-968b-4793-a070-3f609eb0f34d" />

## Setup Instructions
# Deployment Instructions

## Step 1 — Launch EC2 Instance

- Launch Ubuntu EC2 instance in AWS
- Instance type: t3.small

<img width="1366" height="578" alt="ec2-running" src="https://github.com/user-attachments/assets/9491566a-c3ae-4b0e-b4eb-7545025ddd6c" />
- Configure Security Group:
Open ports:
- 22 (SSH)
- 80 (HTTP)
- 3000 (Grafana)
- 5000 (Application)
- 9090 (Prometheus)
- 9100 (Node Exporter)
<img width="1366" height="553" alt="security-group" src="https://github.com/user-attachments/assets/c85ad1f0-d670-4c56-b397-5c1525a0d60f" />

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
<img width="1366" height="577" alt="docker-running" src="https://github.com/user-attachments/assets/893786a5-ebfa-4907-a08f-526e6bed6475" />
<img width="1366" height="685" alt="flask-app" src="https://github.com/user-attachments/assets/9b6fc62a-984b-42dc-b59e-30225730f00d" />

## GitHub Repository
<img width="1366" height="768" alt="ci-cd" src="https://github.com/user-attachments/assets/b3cab741-53aa-4cd4-b0e6-65899aa88321" />
<img width="1366" height="677" alt="ci-cd-git" src="https://github.com/user-attachments/assets/10641f18-a228-42bc-beb2-853904acf5c3" />

##Build Flask Application
docker build -t production-app .
##run container
docker run -d \
--name production-app \
-p 5000:3000 \
production-app
##verify
curl localhost:3001
<img width="1366" height="685" alt="flask-app" src="https://github.com/user-attachments/assets/cfd2fb54-d2fa-4efd-a027-4088ea8c045e" />

##Create Docker Network
docker network create monitoring
##run node exporter
docker run -d \
--name node-exporter \
--network monitoring \
-p 9100:9100 \
prom/node-exporter
<img width="1366" height="683" alt="9100-metrics" src="https://github.com/user-attachments/assets/c56af634-cac4-4741-8341-3ff6f8900869" />

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
<img width="1366" height="685" alt="prometheus-targets" src="https://github.com/user-attachments/assets/50ac9525-b8db-4813-b72d-5238d666480a" />

##Run Grafana
docker run -d \
--name grafana \
--network monitoring \
-p 3000:3000 \
grafana/grafana
##access Grafana
http://<EC2-PUBLIC-IP>:3000
<img width="1366" height="667" alt="grafana-all-metrics" src="https://github.com/user-attachments/assets/93382168-4976-46e4-9fb7-a186a71e9477" />
<img width="1366" height="697" alt="import-node-exporter" src="https://github.com/user-attachments/assets/a166e8e6-9620-43a0-b8cb-d166bb871087" />


##Setup CloudWatch Agent
cd /tmp
wget https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
##create config
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
<img width="1366" height="580" alt="cloudwatch-alarm" src="https://github.com/user-attachments/assets/ede9c8f4-b8ba-4c92-904b-02e9416ab984" />

##Configure GitHub Actions CI/CD
.github/workflows/deploy.yml

