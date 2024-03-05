# Common instructions to set up an ingestion VM

## 1. Set up a Google Cloud VM
```bash
gcloud compute instances create vault-ingest \
    --project=open-targets-genetics-dev \
    --zone=europe-west1-d \
    --machine-type=e2-standard-8 \
    --service-account=234703259993-compute@developer.gserviceaccount.com \
    --scopes=https://www.googleapis.com/auth/cloud-platform \
    --create-disk=auto-delete=yes,boot=yes,device-name=vault-ingest,image=projects/ubuntu-os-cloud/global/images/ubuntu-2204-jammy-v20240228,mode=rw,size=500,type=projects/open-targets-genetics-dev/zones/europe-west1-d/diskTypes/pd-ssd
```

## 2. SSH into the instance
```bash
gcloud compute ssh vault-ingest \
    --project=open-targets-genetics-dev \
    --zone=europe-west1-d
```

## 3. Create a detached virtual terminal
```bash
screen
```

## 4. Install Cloud Storage FUSE
Follow up-to-date instructions in this section: https://cloud.google.com/storage/docs/gcsfuse-quickstart-mount-bucket#install.

## 5. Mount the Gentropy Vault bucket
```bash
mkdir ~/vault
gcsfuse gentropy-vault ~/vault
```
