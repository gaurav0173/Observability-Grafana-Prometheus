# Monitoring Docker Containers with Prometheus & Grafana on EC2

## **Introduction**
This guide provides step-by-step instructions on setting up **Prometheus and Grafana** to monitor **Docker containers** running on an AWS **EC2 instance**.

## **Prerequisites**
- AWS account with access to **EC2**.
- SSH key pair to connect to the EC2 instance.
- Basic knowledge of **Docker** and **Linux commands**.

---

## **Step 1: Launch an EC2 Instance**
1. Go to **AWS Console** → **EC2** → **Launch Instance**.
2. Select an **Ubuntu 20.04/22.04** or **Amazon Linux 2** AMI.
3. Choose an instance type (**t2.micro** for small setups, **t2.medium** for better performance).
4. Configure **Security Groups**:
   - Allow **SSH (22)** (Your IP only)
   - Allow **Prometheus (9090)**
   - Allow **Grafana (3000)**
   - Allow **Node Exporter (9100)**
5. Launch the instance and **connect via SSH**:
   ```sh
   ssh -i your-key.pem ubuntu@your-ec2-public-ip
   ```

---

## **Step 2: Install Docker & Docker-Compose**
Run the following commands on the EC2 instance:
```sh
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

Install Docker Compose:
```sh
sudo apt install docker-compose -y
```

---

## **Step 3: Create a `docker-compose.yml` File**
Create a directory and navigate into it:
```sh
mkdir monitoring && cd monitoring
```

Create a `docker-compose.yml` file:
```sh
nano docker-compose.yml
```

Add the following content:
```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    restart: always

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: always

  node_exporter:
    image: prom/node-exporter
    container_name: node_exporter
    ports:
      - "9100:9100"
    restart: always
```

---

## **Step 4: Configure Prometheus**
Create a `prometheus.yml` file:
```sh
nano prometheus.yml
```

Add the following content:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```

---

## **Step 5: Start Prometheus and Grafana**
Run the following command to start all services:
```sh
docker-compose up -d
```

Verify the containers are running:
```sh
docker ps
```
You should see `prometheus`, `grafana`, and `node_exporter` running.

---

## **Step 6: Access Prometheus and Grafana**
- Open **Prometheus**:  
  👉 `http://<EC2-Public-IP>:9090`
- Open **Grafana**:  
  👉 `http://<EC2-Public-IP>:3000`
  - Default **username**: `admin`
  - Default **password**: `admin`

---

## **Step 7: Configure Grafana to Use Prometheus**
1. Open Grafana (`http://<EC2-Public-IP>:3000`).
2. Go to **Configuration → Data Sources**.
3. Click **Add Data Source** and choose **Prometheus**.
4. Set the **URL** to `http://prometheus:9090` and click **Save & Test**.

---

## **Step 8: Import Grafana Dashboard**
1. In Grafana, go to **Dashboards → Import**.
2. Use **Dashboard ID** `1860` (Node Exporter Full).
3. Select Prometheus as the data source and import.

---

## **Step 9: Verify Metrics**
- Go to **Prometheus UI (`http://<EC2-IP>:9090`)**.
- Run PromQL query: `up`
- In **Grafana**, check the imported dashboard.

---

## **Step 10: Enable Auto-Start on Reboot**
Ensure Docker containers restart on reboot:
```sh
sudo systemctl enable docker
```
To restart the monitoring stack if needed:
```sh
docker-compose down
docker-compose up -d
```

---

## **Conclusion**
You have successfully set up **Prometheus & Grafana** on an **EC2 instance** for monitoring **Docker containers**. This setup allows real-time monitoring, alerting, and visualization of system and application metrics.

---

## **YAML Files**
### **docker-compose.yml**
```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    restart: always

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: always

  node_exporter:
    image: prom/node-exporter
    container_name: node_exporter
    ports:
      - "9100:9100"
    restart: always
```

### **prometheus.yml**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```

