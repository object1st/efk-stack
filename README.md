# EFK Stack Tutorial - Docker Lab Environment

> **Part of the SIEM & Monitoring Blog Series by Geoffrey Burke**

This repository contains everything you need to set up a functional EFK (Elasticsearch, Fluentd, Kibana) test environment using Docker. The blog provides a ready-to-run environment for learning how the EFK stack processes and visualizes log data.

## 🎯 What You'll Learn

- How to deploy a complete EFK stack using Docker Compose
- Configure Fluentd to send Veeam Backup & Replication syslog messages
- Set up syslog forwarding from Veeam to EFK

## 📋 Prerequisites

- **Ubuntu Server VM** (or any Linux environment with Docker support)
- **Docker** and **Docker Compose** installed
- Basic familiarity with command line operations
- *(Optional)* Veeam Backup & Replication for testing syslog forwarding

## 🚀 Quick Start

### 1. Install Docker (if not already installed)

Ubuntu Package


```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Ubuntu Package
```bash
sudo apt update
sudo apt install docker.io
```
Or you can curl the latest version:

```bash
curl -fsSL https://get.docker.com | sh
```
```bash
sudo usermod -aG docker $USER
```
```bash
sudo apt install docker-compose
```


**Important:** Log out and log back in after installation, then verify:

```bash
docker --version
docker compose version
```

### 2. Clone and Run

```bash
git clone https://github.com/object1st/efk-stack.git 
cd efk-stack
docker compose up -d
```

Your EFK stack is now running but we need to setup security

```
# Change elastic user password
docker exec -it elasticsearch bin/elasticsearch-users passwd elastic

# Change kibana_system user password  
docker exec -it elasticsearch bin/elasticsearch-users passwd kibana_system
```
The only passwords that you will be prompted for that matter are the elastic one and kibana_system one.

Add these passwords to the following files kibana.yaml and the fluentd.conf(At the end of the file in the last section). The locations where the passwords are needed are clearly marked.

### 3. Access Kibana

Open your browser and navigate to:
- **If using VM with hostname:** `http://your-vm-hostname:5601`
- **If using IP address:** `http://your-vm-ip:5601`
- **If running locally:** `http://localhost:5601`
Login with your Elastic password

## 📁 Repository Structure

```
efk-stack/
├── README.md                           # This file
├── docker-compose.yml                  # Docker services configuration
├── elasticsearch/
│   └── elasticsearch.yml              # Elasticsearch configuration
└── fluentd/
├── Dockerfile                      # Fluentd custom image build
└── conf/
└── fluent.conf                 # Fluentd pipeline configuration
```

## Configuration Details

### Elasticsearch
- **Port:** 9200
- **Configuration:** Single-node cluster with security disabled (test environment only)
- **Data persistence:** Uses Docker volumes for data storage

### Fluentd
- **TCP Input:** Port 5000 (JSON format)
- **SysLog Input:** Port 5514 (UDP)
- **Pipeline:** Pre-configured to parse Veeam logs and general syslog messages
- **Output:** Sends to Elasticsearch with date-based indices

### Kibana
- **Port:** 5601
- **Configuration:** Auto-connects to Elasticsearch
- **Features:** Full Kibana interface for data exploration and visualization

## 🔧 Testing with Sample Data

### Option 1: Configure Veeam VBR (if available)

1. In Veeam VBR, go to **Menu > Options**
2. Click the **Event Forwarding** tab
3. Add your EFK server:
   - **Server:** Your Docker host IP/hostname
   - **Port:** 5514
   - **Protocol:** UDP

Veeam will automatically send a test message when you save the configuration.

### Option 2: Send Test Syslog Messages

```bash
# Send a test syslog message
echo '<14>$(date --rfc-3339=seconds) your-hostname Veeam.Backup.Manager[1234]: Job [Test Backup Job] completed with Success' | nc -u -w1 your-docker-host 5514

# Send a test JSON message via TCP
echo '{"message": "Test log entry", "level": "info", "timestamp": "'$(date -Iseconds)'"}' | nc your-docker-host 5000
```

## 📊 Exploring Data in Kibana

1. **Access Kibana** at `http://your-host:5601`
2. **Create a Data View:**
   - Go to **Menu > Discover**
   - Click **"Create data view"**
   - Use index pattern: `veeam-logs-*` or `logs-*`
   - Set timestamp field: `@timestamp`
3. **Explore your logs** in the Discover interface

## 🛠️ Troubleshooting

### Container Issues
```bash
# Check container status
docker compose ps

# View logs
docker compose logs elasticsearch
docker compose logs fluentd
docker compose logs kibana

# Restart services
docker compose restart
```

### Common Problems

**Elasticsearch won't start:**
- Check available memory (requires at least 1GB)
- Verify ports 9200 aren't already in use: `netstat -tulpn | grep 9200`

**Fluentd not receiving logs:**
- Verify firewall allows ports 5000 (TCP) and 5514 (UDP)
- Check fluentd logs: `docker compose logs fluentd`

**Kibana connection issues:**
- Ensure Elasticsearch is healthy: `curl http://localhost:9200/_cluster/health`
- Wait for all services to fully start (can take 2-3 minutes)

## 🔄 Reset Environment

To completely reset your environment and start fresh:

```bash
docker compose down --volumes --remove-orphans
docker system prune -a --volumes -f
```

**⚠️ Warning:** This will delete all data, indices, and configurations.

## 🎓 Learning Exercises

Once your environment is running, try these exercises:

1. **Create custom log entries** and observe how Logstash processes them
2. **Build visualizations** in Kibana dashboards
3. **Modify the Fluentd configuration** to parse different log formats
4. **Experiment with Elasticsearch queries** using the Dev Tools console

## 📚 Additional Resources


## ⚠️ Important Notes

- **This is a test environment only** - security is disabled for simplicity
- **Not suitable for production** - lacks authentication, encryption, and proper security configurations
- **Resource usage** - Monitor your system resources as ELK can be memory-intensive

--- 
