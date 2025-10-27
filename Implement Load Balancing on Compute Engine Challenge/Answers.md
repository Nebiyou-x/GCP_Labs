#!/bin/bash

ZONE="us-east1-d"
REGION="us-east1"

## TASK 1



# Create VM instances
gcloud compute instances create web1 \
--zone=$ZONE \
--machine-type=e2-small \
--tags=network-lb-tag \
--image-family=debian-11 \
--image-project=debian-cloud \
--metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web1</h3>" | tee /var/www/html/index.html'

gcloud compute instances create web2 \
--zone=$ZONE \
--machine-type=e2-small \
--tags=network-lb-tag \
--image-family=debian-11 \
--image-project=debian-cloud \
--metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web2</h3>" | tee /var/www/html/index.html'

gcloud compute instances create web3 \
--zone=$ZONE \
--machine-type=e2-small \
--tags=network-lb-tag \
--image-family=debian-11 \
--image-project=debian-cloud \
--metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web3</h3>" | tee /var/www/html/index.html'

# Create firewall rule for HTTP
gcloud compute firewall-rules create www-firewall-network-lb --allow tcp:80 --target-tags network-lb-tag

## TASK 2


# Reserve a static external IP
gcloud compute addresses create network-lb-ip-1 --region=$REGION

# Create health check
gcloud compute http-health-checks create basic-check

# Create target pool and add instances
gcloud compute target-pools create www-pool \
    --region=$REGION \
    --http-health-check basic-check

gcloud compute target-pools add-instances www-pool \
    --instances web1,web2,web3 --zone=$ZONE

# Create forwarding rule
gcloud compute forwarding-rules create www-rule \
    --region=$REGION \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool

# Get the load balancer IP address
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region=$REGION  --format="json" | jq -r .IPAddress)








## Task 3 

# Create instance template
gcloud compute instance-templates create lb-backend-template \
   --region=$REGION \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | tee /var/www/html/index.html
     systemctl restart apache2'

# Create managed instance group
gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=$ZONE

# Allow health check traffic
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80

# Reserve global static IP
gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global

# Create HTTP health check
gcloud compute health-checks create http http-basic-check --port 80

# Create backend service and attach instance group
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global

gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=$ZONE \
  --global

# Create URL map and HTTP proxy
gcloud compute url-maps create web-map-http --default-service web-backend-service
gcloud compute target-http-proxies create http-lb-proxy --url-map web-map-http

# Create forwarding rule
gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1 \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
