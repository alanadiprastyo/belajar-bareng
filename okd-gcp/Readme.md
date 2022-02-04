# Belajar Bareng - Deploy OKD on GCP

1. membuat instance untuk jump host - CentOS 7/8

2. install gcloud cli
```
curl https://sdk.cloud.google.com | bash
```
3. initialize gcloud
```
gcloud init
```
Note: Login via gmail

4. create project for okd
```
# Set some variables first:
export project=my-okd-demo
export my_billing=$YOUR_GOOGLE_BILLING_ID

# Create project
gcloud projects create --set-as-default $project

# Link billing account
gcloud alpha billing projects link $project --billing-account="$my_billing"
```
Note: untuk billing bisa enable di dashboard

5. Enable GCP API to use during installation OKD
```
for api in compute.googleapis.com cloudapis.googleapis.com cloudresourcemanager.googleapis.com dns.googleapis.com iamcredentials.googleapis.com iam.googleapis.com servicemanagement.googleapis.com serviceusage.googleapis.com storage-api.googleapis.com storage-component.googleapis.com; do
  gcloud services enable $api
done
```

6. Setup DNS Public
```
# Set up a delegated DNS zone
gcloud dns managed-zones create cloudnusantara --dns-name 'cloudnusantara.my.id' --description 'cloud nusantara for okd'
```
Note: change NS domain hosting to NS google cloud
```
    ns-cloud-b1.googledomains.com.
    ns-cloud-b2.googledomains.com.
    ns-cloud-b3.googledomains.com.
    ns-cloud-b4.googledomains.com. 
```
Note: sesuaikan pada NS GCP masing2

7. Create Service Account
```
gcloud iam service-accounts create initial-okd-sa --description="Initial SA for OKD" --display-name="OKD SA"
```

8. Assign permission to SA
```
# Check [1] for a more appropriate list of permissions for production clusters
gcloud projects add-iam-policy-binding $project --member="serviceAccount:initial-okd-sa@$project.iam.gserviceaccount.com" --role="roles/owner"
```

9. Create a key file for the service account 
```
gcloud iam service-accounts keys create initial-okd-sa.json --iam-account=initial-okd-sa@$project.iam.gserviceaccount.com
```

10. Verify Quota on your project
- pastikan minimal Persistant storage 1TB dan vcpu 40

11. Deploy Cluster OKD
```
mkdir -p ~/.gcp
cp initial-sa-okd.json .gcp/osServiceAccount.json

mkdir okdtest && cd okdtest
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
wget https://github.com/openshift/okd/releases/download/4.9.0-0.okd-2022-01-29-035536/openshift-install-linux-4.9.0-0.okd-2022-01-29-035536.tar.gz

tar -zxvf openshift-install-linux-4.9.0-0.okd-2022-01-29-035536.tar.gz openshift-client-linux.tar.gz

# Define install-config
./openshift-install create install-config
----
? SSH Public Key /home/alan/.ssh/id_rsa.pub
? Platform gcp
INFO Credentials loaded from file "/home/alan/.gcp/osServiceAccount.json" 
? Project ID my-okd-demo (my-okd-demo)
? Region asia-southeast2
? Base Domain cloudnusantara.my.id
? Cluster Name okd
? Pull Secret [? for help] ******
? Pull Secret [? for help] **********************************
----

# Custom install-config yaml type platform

cat install-config.yaml
----
apiVersion: v1
baseDomain: cloudnusantara.my.id
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform:
    gcp:
      type: e2-standard-4
  replicas: 2
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform:
    gcp:
      type: e2-standard-4
  replicas: 3
metadata:
  creationTimestamp: null
  name: okd
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OpenshiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  gcp:
    projectID: my-okd-demo
    region: asia-southeast2
publish: External
pullSecret: '{"auths":{"fake":{"auth": "bar"}}}'
sshKey: |
  ssh-rsa ssh-key......

# create cluster
./openshift-install create cluster --dir .
```

12. Destroy Cluster
```
./openshift-install destroy cluster --dir .
```

Ref: 
- https://docs.okd.io/4.9/installing/installing_gcp/installing-gcp-default.html#installing-gcp-default
- https://docs.okd.io/4.9/installing/installing_gcp/installing-gcp-customizations.html#installation-gcp-config-yaml_installing-gcp-customizations
- https://dustymabe.com/2020/10/07/gcp-quickstart-guide-for-openshift-okd/