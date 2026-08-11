# 📊 Grafana Monitoring, PromQL & Dashboard Cheat Sheet

> [!IMPORTANT]
> **Quick Access Credentials & Port**
> - **Default Access URL:** `http://localhost:3000`
> - **Default Username:** `admin` | **Default Password:** `admin`
> - **Node Exporter Pre-built Dashboard ID:** `1860`

---

## ⚡ 1. Service Management & CLI Commands

| Action | Linux / Systemd Command | Docker Command |
| :--- | :--- | :--- |
| **Start Service** | `sudo systemctl start grafana-server` | `docker run -d -p 3000:3000 --name=grafana grafana/grafana` |
| **Stop Service** | `sudo systemctl stop grafana-server` | `docker stop grafana` |
| **Check Status** | `sudo systemctl status grafana-server` | `docker ps -a` |
| **Install Plugin** | `grafana-cli plugins install <plugin-id>` | `docker exec -it grafana grafana-cli plugins install <plugin-id>` |

---

## 🔌 2. Common Data Sources & Default Ports

```yaml
Prometheus:  http://localhost:9090  # (Primary Time-Series DB)
Loki:        http://localhost:3100  # (Log Aggregation Engine)
InfluxDB:    http://localhost:8086  # (Time-Series DB)
Elastic:     http://localhost:9200  # (Search & Analytics)
Graphite:    http://localhost:2003  # (Real-time Metrics)
📈 3. Essential PromQL Query Formulas[!TIP]Use these exact formulas inside your Grafana panel query editor when using Prometheus as a Data Source.💻 System Performance Metrics🟢 CPU Usage (%)Code snippet100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
🟡 Memory Usage (%)Code snippet(100 - ((node_memory_MemAvailable_bytes * 100) / node_memory_MemTotal_bytes))
🔴 Disk Space Usage (%)Code snippet100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})
🌐 Network & HTTP Traffic MetricsCode snippet# Network Receive Rate (Bytes/sec)
rate(node_network_receive_bytes_total[5m])

# Network Transmit Rate (Bytes/sec)
rate(node_network_transmit_bytes_total[5m])

# Total HTTP Request Rate (Requests/sec)
rate(http_requests_total[5m])

# HTTP 5xx Server Error Rate
rate(http_requests_total{status=~"5.."}[5m])
🎯 4. Visual Panel Types & Selection Guide[!NOTE]Select visual panel types based on data representation requirements:Time Series: Line/bar charts for continuous time-based metrics (CPU, RAM, Traffic).Stat: Single large callout numbers for real-time state indicators.Gauge: Speedometer visual with custom color thresholds (Green / Yellow / Red).Bar Chart: Categorical data comparison.Table: Structured data rows, status logs, and multi-field outputs.Logs (Loki): Real-time log stream tailing and text searches.🔀 5. Dynamic Dashboard VariablesTo create dynamic multi-environment dashboards, set up variables under Dashboard Settings ➔ Variables:Variable Name: instanceType: QueryData Source: PrometheusQuery Definition: label_values(node_cpu_seconds_total, instance)Panel Query Reference:Code snippetnode_memory_MemAvailable_bytes{instance="$instance"}
🚨 6. Alerting Pipeline & Architecture[ Data Query (A) ] ➔ [ Reduce to Last Value (B) ] ➔ [ Condition: C > 80% ] ➔ [ Notification Alert ]
Pending Buffer (For): Set to 5m to filter out temporary transient spikes.Alert Notification Channels: Discord Webhooks, Slack, PagerDuty, Email, Telegram.📌 7. Lab Execution & Workflow SummaryTask / ModuleConfiguration DetailsStatusGrafana SetupServer listening on port 3000Operational ✅Prometheus BindingConnected via local endpoint on port 9090Active ✅System DashboardsConfigured CPU, Memory, Disk, and Network panelsConfigured ✅Alerting EngineThreshold set for >80% resource usageMonitoring ✅
