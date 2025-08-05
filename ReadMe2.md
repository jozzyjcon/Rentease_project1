# Monitoring GitHub Actions with Prometheus and Grafana

This serves as project 2, a continuation of project 1, which is the base project. 
It demonstrates how to monitor GitHub Actions workflows using..
**Prometheus, Grafana, and the github-actions-exporter.** 

## Benefits
The goal is to visualize key CI/CD metrics like:

1. Build Duration
2. Failure rates
3. Test coverage


Reporting, visualizing, and monitoring are ideal for DevOps engineers, SREs, and security professionals 
looking to enhance visibility over CI/CD performance. It gives you a quick view to carry out your
gap analysis better in optimizing your workflow. Let's jump right into it.

 ## **Architecture Overview**
 Here's a high-level architecture diagram of the setup
![GitHub Actions](screenshots/Monitoring%20CICD%20Architecture%20with%20prometheus%20and%20Grafana.png)

## Prerequisites
1. GitHub repository with existing workflows
2. Access to a server to run Prometheus, Grafana, and GitHub Actions Exporter
3. Ensure the following ports are open
![GitHub Actions](/screenshots/1_After_EC2_Enabled%20ports.png)

 ## Step-by-step Guide

***Install Docker Compose***
```bash 
sudo apt update
sudo apt install  docker-compose -y
sudo usermod -aG docker $USER
newgrp docker  # Refresh group permissions
```

***Verify Docker and Docker-Compose are running***
```bash
docker --version && docker-compose --version
```
![Github Actions](screenshots/00Docker%20running.png)

***Create the folder*** `monitoring` with two files `docker-compose.yml` and  prometheus.yml in it.
However, the entire file structure of our project appears as follows.

```bash
monitoring/
├── docker-compose.yml
├── prometheus.yml
└── github_exporter/
    └── config.yaml
```

`docker-compose.yml` configuration code
```bash
version: '3.8'  # Add this line at the top
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    restart: unless-stopped

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    restart: unless-stopped

```

`prometheus.yml` configuration below:
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

```
***Start the services***
```bash
cd ~/monitoring
docker-compose up -d
```
Verify Prometheus is running on port `9090` 
![](screenshots/3_Prom%20interface.png)

***Install GitHub Exporter***
We'll use the ghcr.io/labbs/github-actions-exporter image
Create another folder with a configuration file `/github-exporter/config.yaml`

```bash
github:
  token: "ghp_yourNewTokenHere"  # Create at: https://github.com/settings/tokens (repo + admin(read):org + workflow scopes)
  repos:
    - owner: "jozzyjcon"
      name: "Rentease_project1"
```
***Update your docker-compose.yml***
```bash

  github-exporter:  # Properly indented under services
    image: ghcr.io/labbs/github-actions-exporter:latest
    ports:
      - "9999:9999"
    volumes:
      - /home/ubuntu/monitoring/github-exporter/config.yaml:/config.yaml  # Full absolute path
    restart: unless-stopped
```

***Update*** `prometheus.yml`
```bash
- job_name: 'github_actions'
  static_configs:
    - targets: ['github-exporter:9999']
```

***Restart services***
```bash
docker-compose down && docker-compose up -d
```
***Confirm Prometheus, Grafana, and Github-actions-exporter docker containers***

![](screenshots/10_The%20only%20three%20docker%20images%20I%20need.png)

***Configure Grafana***
Access Grafana at `http://your-server-ip:3000`
Login: `admin / admin` (change password when prompted)
Verify Grafana is running on port `9090`
![](screenshots/4_Grafana%20interface.png)

***Add Prometheus datasource***
Collections → Data Sources → Add Prometheus
URL: `http://<server_IP>:9090`

***Create a new dashboard.*** Ensure to choose Prometheus as the Datasource
![Github Actions](screenshots/screenshots/13_Add_Dashboard.png)

Add your desired metric and configure using the most suitable graphical representations for your GitHub Actions workflows
