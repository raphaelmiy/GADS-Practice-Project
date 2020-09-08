# Translations

In this lab, we create several VPC networks and VM instances and test connectivity across networks.

Create managementnet VPC Network

```bash
gcloud compute networks create managementnet --project=qwiklabs-gcp-02-1079938b2afa --subnet-mode=custom --bgp-routing-mode=regional

gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-02-1079938b2afa --range=10.130.0.0/20 --network=managementnet --region=us-central1
```

Create privatenet VPC Network

```bash
gcloud compute networks create privatenet --subnet-mode=custom

gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

gcloud compute networks list

gcloud compute networks subnets list --sort-by=NETWORK
```

Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network.

```bash
gcloud compute --project=qwiklabs-gcp-02-1079938b2afa firewall-rules create managementnet-allow-icmp-ssh-rdp --description="Firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network." --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```

Create the firewall rules for privatenet

```bash
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

gcloud compute firewall-rules list --sort-by=NETWORK
```

Create the managementnet-us-vm instance

```bash
gcloud beta compute --project=qwiklabs-gcp-02-1079938b2afa instances create managementnet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=managementsubnet-us --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=130724237806-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=managementnet-us-vm --reservation-affinity=any
```

Create the privatenet-us-vm instance

```bash
gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us

gcloud compute instances list --sort-by=ZONE
```

Ping the external IP addresses

```bash
ping -c 3 34.78.48.59
ping -c 3 35.222.75.35
ping -c 3 34.121.233.251
```

Create the VM instance with multiple network interfaces

```bash
gcloud beta compute --project=qwiklabs-gcp-02-1079938b2afa instances create vm-appliance --zone=us-central1-c --machine-type=n1-standard-4 --subnet=privatesubnet-us --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=130724237806-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=vm-appliance --reservation-affinity=any
```
