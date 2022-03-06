# README

This repo shows how to use Ansible to deploy Ops-Agent to GCP VMs

## References
https://github.com/GoogleCloudPlatform/google-cloud-ops-agents-ansible/tree/master/tutorial


https://cloud.google.com/blog/products/operations/use-ansible-to-deploy-the-google-cloud-ops-agent

Related Youtube video
https://www.youtube.com/watch?v=GQgNygd-XJU

SSH through IAP 
https://binx.io/blog/2021/03/10/how-to-tell-ansible-to-use-gcp-iap-tunneling/


## Install Ansible
```
sudo apt-get install python3-setuptools
pip install -U pip setuptools 
python -m pip install --user ansible
pip install requests google-auth

# add ~/.local/bin to your PATH
```

## Create Service Account that has IAP required permissions 

```
export PROJECT_ID="your project id"
export SERVICE_ACCOUNT_ID=ansible-sa

gcloud config set project $PROJECT_ID
gcloud iam service-accounts create $SERVICE_ACCOUNT_ID --description="Service account for ansible tutorial" --display-name="ansible-sa"

# IAP requires these 3 roles
# Compute Instance Admin (v1)   roles/compute.instanceAdmin.v1
# IAP-Secured Tunnel User       roles/iap.tunnelResourceAccessor
# Service Account User          roles/iam.serviceAccountUser


gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com" --role="roles/iam.serviceAccountUser"

gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com" --role="roles/compute.instanceAdmin.v1"

gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com" --role="roles/iap.tunnelResourceAccessor"

# Generate key files
gcloud iam service-accounts keys create key-file --iam-account=$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com

export GCP_SERVICE_ACCOUNT_FILE=$PWD/key-file

sed "s/PROJECT_ID/$PROJECT_ID/g" inventory.gcp.yaml.template > inventory.gcp.yaml
sed -i "s/KEY_FILE/key-file/g" inventory.gcp.yaml

```

## Test ansible and deploy ops-agent
```
ansible-inventory all -i inventory.gcp.yaml --list
ansible-inventory --graph 

# install galaxy role
ansible-galaxy install git+https://github.com/GoogleCloudPlatform/google-cloud-ops-agents-ansible.git

# run playbook
ansible-playbook example_playbook.yaml -i inventory.gcp.yaml
```

## Use the instruction below if you don't use IAP

If you create a GCP VM which has direct connection to the target VMs  
You don't need to use IAP.

Here are the steps to use direct SSH connections.

Use the files in `ansible-without-iap`

```
# add your ssh private key to ssh-agent
# This will be generate the first time you use IAP
# or generate the key pair and manually copy the public key to the target server
eval `ssh-agent -s`
ssh-add -l
ssh-add ~/.ssh/google_compute_engine

ansible-inventory --graph
ansible-galaxy install git+https://github.com/GoogleCloudPlatform/google-cloud-ops-agents-ansible.git
ansible-playbook example_playbook.yaml
```

NOTE:  
If host is in a different zone then change the inventory.gcp.yaml to use private IP  
This is because GCP FQDN includes zone  
```
  ansible_host: networkInterfaces[0].networkIP
  # ansible_host: name
```