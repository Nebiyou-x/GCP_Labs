# 🌩️ Google Cloud Load Balancing Challenge

## 📘 Scenario

You are a **junior cloud engineer** tasked with providing network functionality to Compute Engine virtual machine (VM) instances on a Google Cloud Virtual Private Cloud (VPC) network.  

Because you cannot create VM instances, containers, or App Engine applications without a VPC network, each Google Cloud project has a **default network** configured to get you started.

To support **load balancing** for the network traffic, you must understand the difference between a **Network Load Balancer (TCP/UDP)** and an **HTTP(S) Load Balancer**, and configure both for your applications running on Compute Engine VMs.

---

## 🎯 Your Challenge

You are required to:

1. **Create multiple web server instances** with firewall rules.  
2. **Configure a Network Load Balancer**.  
3. **Create an HTTP Load Balancer**.

All resources should be created in the following region and zone unless otherwise specified:

| Property | Value |
|-----------|--------|
| **Region** | `us-east1` |
| **Zone** | `us-east1-d` |

---

## 🧩 Task 1: Create Multiple Web Server Instances

Create **three Compute Engine VM instances** with Apache installed.  
All instances should be under the **default network**.

### Configuration

| Property | Value |
|-----------|--------|
| VM Instance - 1 | `web1` |
| VM Instance - 2 | `web2` |
| VM Instance - 3 | `web3` |
| Machine Type | `e2-small` |
| Tags | `network-lb-tag` |
| Image Family | `debian-12` |
| Image Project | `debian-cloud` |

Use the following startup script (modify `<num>` to match each VM name):

```bash
#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web<num></h3>" | tee /var/www/html/index.html




⚙️ Task 2: Configure the Network Load Balancing Service

You will now configure a Network Load Balancer (NLB) to distribute incoming traffic to your web servers.

Configuration
Property	Value
Static External IP	network-lb-ip-1
Target Pool	www-pool
Ports	80


☁️ Task 3: Create an HTTP Load Balancer

Now create a global HTTP Load Balancer using a managed instance group as the backend.

Configuration
Property	Value
Backend Template	lb-backend-template
Managed Instance Group	lb-backend-group
Machine Type	e2-medium
Image Family / Project	debian-11, debian-cloud
Firewall Rule	fw-allow-health-check
Source Ranges	130.211.0.0/22, 35.191.0.0/16
Port	80
External IP	lb-ipv4-1
Health Check	http-basic-check
URL Map	web-map-http
Target HTTP Proxy	http-lb-proxy