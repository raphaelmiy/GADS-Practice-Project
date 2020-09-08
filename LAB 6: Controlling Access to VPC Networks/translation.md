# Translations for Controlling Access to VPC Networks Lab

This is the code translation for the Controlling Access to VPC Networks lab.

## Task at hand

In this lab, you create two nginx web servers and control external HTTP access to the web servers using tagged firewall rules. Then you explore IAM roles and service accounts.

## Objectives

- Create an nginx web server
- Create tagged firewall rules
- Create a service account with IAM roles
- Explore permissions for the Network Admin and Security Admin roles

### Create the blue server with a network tag

```bash
gcloud beta compute --project=qwiklabs-gcp-04-233c7cbee659 instances create blue --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=651018344613-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=web-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=blue --reservation-affinity=any
```

### Create the green server

```bash
gcloud beta compute --project=qwiklabs-gcp-04-233c7cbee659 instances create green --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=651018344613-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=green --reservation-affinity=any
```

### Install nginx and customize the welcome page (Blue Server)

```bash
sudo apt-get install nginx-light -y
sudo nano /var/www/html/index.nginx-debian.html
```

Replace the Welcome to nginx! line with Welcome to the blue server!
Press CTRL+O, ENTER, CTRL+X.

### Install nginx and customize the welcome page (Green Server)

```bash
sudo apt-get install nginx-light -y
sudo nano /var/www/html/index.nginx-debian.html
```

Replace the Welcome to nginx! line with Welcome to the green server!
Press CTRL+O, ENTER, CTRL+X.

### Create the tagged firewall rule

```bash
gcloud compute --project=qwiklabs-gcp-04-233c7cbee659 firewall-rules create allow-http-web-server --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=web-server
```

### Create a test-vm

```bash
gcloud compute instances create test-vm --machine-type=n1-standard-1 --subnet=default --zone=us-central1-a
```

### Test HTTP connectivity

```bash
curl 10.128.0.2
curl 10.128.0.3
curl 35.194.12.156
curl 34.70.48.15
```

### Verify current permissions

```bash
gcloud compute firewall-rules list
gcloud compute firewall-rules delete allow-http-web-server
```
