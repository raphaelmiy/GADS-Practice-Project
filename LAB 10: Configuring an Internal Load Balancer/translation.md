# Translations for Configuring an Internal Load Balancer lab

## Objectives

- Create internal traffic and health check firewall rules
- Create a NAT configuration using Cloud Router
- Configure two instance templates
- Create two managed instance groups
- Configure and test an internal load balancer

### Configure internal traffic and health check firewall rules

Configure firewall rules to allow internal traffic connectivity from sources in the 10.10.0.0/16 range. This rule allows incoming traffic from any client located in the subnet.

#### Create the firewall rule to allow traffic from any sources in the 10.10.0.0/16 range

```bash
gcloud compute --project=qwiklabs-gcp-04-4860f8feec44 firewall-rules create fw-allow-lb-access --description="Create a firewall rule to allow traffic in the 10.10.0.0/16 subnet." --direction=INGRESS --priority=1000 --network=my-internal-app --action=ALLOW --rules=all --source-ranges=10.10.0.0/16 --target-tags=backend-service
```

#### Create the health check rule

Create a firewall rule to allow health checks.

```bash
gcloud compute --project=qwiklabs-gcp-04-4860f8feec44 firewall-rules create fw-allow-health-checks --description="Create a firewall rule to allow health checks." --direction=INGRESS --priority=1000 --network=my-internal-app --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=backend-service
```

### Configure instance templates and create instance groups

A managed instance group uses an instance template to create a group of identical instances. Use these to create the backends of the internal load balancer.

#### Configure the instance templates

```bash
gcloud beta compute --project=qwiklabs-gcp-04-4860f8feec44 instance-templates create instance-template-1 --machine-type=e2-medium --subnet=projects/qwiklabs-gcp-04-4860f8feec44/regions/us-central1/subnetworks/subnet-a --no-address --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --maintenance-policy=MIGRATE --service-account=83479686391-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=backend-service --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-template-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

#### Create the managed instance groups

```bash
gcloud compute --project=qwiklabs-gcp-04-4860f8feec44 instance-groups managed create instance-group-1 --base-instance-name=instance-group-1 --template=instance-template-1 --size=1 --zone=us-central1-a

gcloud beta compute --project "qwiklabs-gcp-04-4860f8feec44" instance-groups managed set-autoscaling "instance-group-1" --zone "us-central1-a" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"

gcloud compute --project=qwiklabs-gcp-04-4860f8feec44 instance-groups managed create instance-group-2 --base-instance-name=instance-group-2 --template=instance-template-2 --size=1 --zone=us-central1-b

gcloud beta compute --project "qwiklabs-gcp-04-4860f8feec44" instance-groups managed set-autoscaling "instance-group-2" --zone "us-central1-b" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"
```

#### Verify the backends

Verify that VM instances are being created in both subnets and create a utility VM to access the backends' HTTP sites.

```bash
gcloud beta compute --project=qwiklabs-gcp-04-4860f8feec44 instances create utility-vm --zone=us-central1-f --machine-type=f1-micro --subnet=subnet-a --private-network-ip=10.10.20.50 --no-address --maintenance-policy=MIGRATE --service-account=83479686391-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=utility-vm --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```
