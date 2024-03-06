# UKB PPP

Mirror into `gs://gentropy-vault/ukb-ppp` started on 2024-02-29 and completed on 2024-03-01.

## 1. Request access to dataset
Go to the dataset hosted on the Synapse platform: https://www.synapse.org/#!Synapse:syn51364943/files/. If you don't have access to it, you'll need to request it from the platform. Once given, this access does not expire.

## 2. Obtain programmatic access token
1. Go to https://www.synapse.org/#!PersonalAccessTokens.
2. Create a new token with “View” and “Download” permissions.

This token is  valid for 180 days. After this, you will need to repeat this section to generate a new one.

## 3. Create the dataset directory
```bash
cd ~/vault
mkdir ukb-ppp
cd ukb-ppp
```

## 4. Install the Synapse client
```bash
sudo apt install -y python3-pip
pip install synapseclient
export PATH=$PATH:~/.local/bin
```

## 5. Sync the data
```bash
time synapse get -r syn51364943 
```

When prompted for username, leave it blank. When prompted for authentication token, enter the one you obtained during step 2.
