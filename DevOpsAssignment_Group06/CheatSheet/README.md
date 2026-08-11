# 📊 Grafana & PromQL Cheat Sheet

> [!IMPORTANT]
> **Quick Access Credentials**
> - **URL:** `http://localhost:3000`
> - **User:** `admin` | **Pass:** `admin`
> - **Node Exporter ID:** `1860`

---

## ⚡ 1. Commands & Management

### Linux Service
```bash
# Start Server
sudo systemctl start grafana-server

# Check Status
sudo systemctl status grafana-server

Docker Commands
Bash
# Run Grafana Container
docker run -d -p 3000:3000 \
  --name=grafana grafana/grafana

# Install Plugin
grafana-cli plugins install <plugin-id>
🔌 2. Data Sources & Ports
YAML
Prometheus: http://localhost:9090
Loki:       http://localhost:3100
InfluxDB:   http://localhost:8086
Elastic:    http://localhost:9200
📈 3. Key PromQL Formulas
[!TIP]
Use these formulas inside the Query Editor.

💻 System Metrics
🟢 CPU Usage (%)

Code snippet
100 - (avg by (instance) 
(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
🟡 Memory Usage (%)

Code snippet
(100 - ((node_memory_MemAvailable_bytes * 100) / 
node_memory_MemTotal_bytes))
🔴 Disk Space Usage (%)

Code snippet
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / 
node_filesystem_size_bytes{mountpoint="/"})
🌐 Network & Traffic
Code snippet
# Network Receive Rate (Bytes/sec)
rate(node_network_receive_bytes_total[5m])

# HTTP Request Rate
rate(http_requests_total[5m])

# HTTP 5xx Server Errors
rate(http_requests_total{status=~"5.."}[5m])
🎯 4. Panel Selection Guide
Time Series: Line graphs for continuous metrics (CPU, RAM).

Stat: Single large numbers for real-time values.

Gauge: Speedometer visual with Green/Yellow/Red zones.

Table: Structured data rows and multi-field outputs.

Logs: Real-time log streaming via Loki.

🔀 5. Dynamic Dashboard Variables
Path: Dashboard Settings ➔ Variables

Name: instance

Type: Query

Query: label_values(node_cpu_seconds_total, instance)

Panel Reference:

Code snippet
node_memory_MemAvailable_bytes{instance="$instance"}
🚨 6. Alert Pipeline
Plaintext
[ Query (A) ] 
   └── [ Reduce (B) ] 
          └── [ Condition: C > 80% ] 
                 └── [ Send Alert ]
📌 7. Lab Overview
Grafana Server: Port 3000 (Active ✅)

Prometheus Source: Port 9090 (Connected ✅)

System Panels: CPU/RAM/Disk (Configured ✅)

Alert Engine: Threshold >80% (Active ✅)
