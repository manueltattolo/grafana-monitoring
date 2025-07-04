# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  # Port forwarding
  config.vm.network "forwarded_port", guest: 5000, host: 5100, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 8000, host: 8100, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 9090, host: 9090, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 3000, host: 3100, host_ip: "127.0.0.1"

  config.vm.provision "shell", inline: <<-SHELL
    set -e
    # VAGRANT SETUP
    apt-get update
    apt-get install -y python3-pip python3-venv git curl apt-transport-https software-properties-common wget gnupg

    # PY APP SETUP
    python3 -m venv /opt/myenv
    /opt/myenv/bin/pip install --upgrade pip
    /opt/myenv/bin/pip install flask prometheus_client

    mkdir -p /opt/myapp
    cat <<'PY' > /opt/myapp/app.py
from flask import Flask
from prometheus_client import start_http_server, Counter
import logging

logging.basicConfig(level=logging.INFO)

REQUEST_COUNT = Counter("http_requests_total", "Total HTTP requests")

app = Flask(__name__)

@app.route("/")
def index():
    REQUEST_COUNT.inc()
    logging.info("Received request at /")
    return "Hello darling!"

if __name__ == "__main__":
    start_http_server(8000)
    app.run(host="0.0.0.0", port=5000)
PY

    if ! grep -q "/opt/myenv/bin/python /opt/myapp/app.py" /home/vagrant/.bashrc; then
      echo "/opt/myenv/bin/python /opt/myapp/app.py &" >> /home/vagrant/.bashrc
    fi

    # PROMETHEUS APP SETUP
    cd /opt
    curl -LO https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-arm64.tar.gz
    tar -xzf prometheus-2.52.0.linux-arm64.tar.gz
    mv prometheus-2.52.0.linux-arm64 prometheus
    rm prometheus-2.52.0.linux-arm64.tar.gz
    mkdir -p /opt/prometheus/data
    chown vagrant:vagrant /opt/prometheus/data

    cat <<'YML' > /opt/prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'python-app'
    static_configs:
      - targets: ['localhost:8000']
YML

    if ! grep -q "/opt/prometheus/prometheus" /home/vagrant/.bashrc; then
      echo "/opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml --web.listen-address=\":9090\" &" >> /home/vagrant/.bashrc
    fi

    # GRAFANA FROM DOCS
    echo "Installing Grafana..."
    mkdir -p /etc/apt/keyrings
    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | tee /etc/apt/sources.list.d/grafana.list > /dev/null
    apt-get update
    apt-get install -y grafana
    systemctl enable grafana-server
    systemctl start grafana-server
  SHELL
end
