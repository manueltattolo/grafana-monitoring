# Grafana Monitoring

## Ubuntu Vagrant Dev Stack (ARM64 - Apple Silicon)

This Vagrant setup creates an Ubuntu 24.04 VM configured with:

- A Flask app exposing Prometheus metrics  
- Prometheus scraping those metrics  
- Grafana for visualizing the data  

**Note:** This setup is designed for **ARM64 architecture**, specifically Macs with Apple Silicon (M1/M2/M3).  
It uses the `bento/ubuntu-24.04` Vagrant box compatible with ARM64 and runs on **VirtualBox**.  
If you're using an x86_64 (Intel/AMD) system, you'll need to choose a different Vagrant box compatible with your architecture.

## Quick Start
```bash
vagrant up
```

Once provisioned, services will be available at:
Flask app → http://localhost:5100
Prometheus → http://localhost:9090
Grafana → http://localhost:3100 (login: admin / admin)

