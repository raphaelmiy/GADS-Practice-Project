# Translation For Configuring an HTTP Load Balancer with Cloud Armor Lab

## Objectives

- Create a health check firewall rule
- Create two regional NAT configurations using Cloud Router
- Configure two instance templates
- Create two managed instance groups
- Configure an HTTP load balancer with IPv4 and IPv6
- Stress test an HTTP load balancer
- Deny an IP address to restrict access to an HTTP load balancer

### Create the health check rule

```bash
gcloud compute --project=qwiklabs-gcp-04-9f1713a59cec firewall-rules create fw-allow-health-checks --description="Create a firewall rule to allow health checks." --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-checks
```

### Configure an instance template and create instance groups

#### Configure the instance template

```bash
gcloud beta compute --project=qwiklabs-gcp-04-9f1713a59cec instance-templates create us-central1-template --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-04-9f1713a59cec/regions/us-central1/subnetworks/default --no-address --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh --maintenance-policy=MIGRATE --service-account=32101116113-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=allow-health-checks --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=us-central1-template --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

#### Create the managed instance groups

Create a managed instance group in us-central1 and one in europe-west1

```bash
gcloud beta compute --project=qwiklabs-gcp-04-9f1713a59cec instance-groups managed create us-central1-mig --base-instance-name=us-central1-mig --template=us-central1-template --size=1 --zones=us-central1-b,us-central1-c,us-central1-f --instance-redistribution-type=PROACTIVE

gcloud beta compute --project "qwiklabs-gcp-04-9f1713a59cec" instance-groups managed set-autoscaling "us-central1-mig" --region "us-central1" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"
```

```bash
gcloud beta compute --project=qwiklabs-gcp-04-9f1713a59cec instance-groups managed create europe-west1-mig --base-instance-name=europe-west1-mig --template=europe-west1-template --size=1 --zones=europe-west1-b,europe-west1-c,europe-west1-d --instance-redistribution-type=PROACTIVE

gcloud beta compute --project "qwiklabs-gcp-04-9f1713a59cec" instance-groups managed set-autoscaling "europe-west1-mig" --region "europe-west1" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"
```

#### Stress test the HTTP load balancer

```bash
gcloud beta compute --project=qwiklabs-gcp-04-9f1713a59cec instances create siege-vm --zone=us-west1-c --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=32101116113-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=siege-vm --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

#### Create the security policy

```bash
gcloud compute --project=qwiklabs-gcp-04-9f1713a59cec security-policies create deny-siege

gcloud compute --project=qwiklabs-gcp-04-9f1713a59cec security-policies rules create 1000 --action=deny\(403\) --security-policy=deny-siege --src-ip-ranges=35.233.246.31

gcloud compute --project=qwiklabs-gcp-04-9f1713a59cec security-policies rules create 2147483647 --action=allow --security-policy=deny-siege --description="Default rule, higher priority overrides it" --src-ip-ranges=\*

gcloud compute --project=qwiklabs-gcp-04-9f1713a59cec backend-services update http-backend --security-policy=deny-siege
```
