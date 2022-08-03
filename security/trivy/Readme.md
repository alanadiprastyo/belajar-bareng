# Belajar Bareng - Trivy Scan Contaier Image

## List Materi Belajar

- Install Trivy
- Quick start trivy


## Mulai Belajar

1. Install Trivy

RHEL/CentOS
```
$ sudo vim /etc/yum.repos.d/trivy.repo
[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/$releasever/$basearch/
gpgcheck=0
enabled=1
$ sudo yum -y update
$ sudo yum -y install trivy
```

Ubuntu
```
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

Ref : https://aquasecurity.github.io/trivy/v0.22.0/getting-started/installation/

2. Scan Image Container

```bash
trivy image [YOUR_IMAGE_NAME]
```
List Image => 
- python:3.4-alpine
- alpine:3.10
- quay.io/alanadiprastyo/go-sampel

3. Scan Image Tar Files

```
docker save [Image_contianer] -o [Output_file.tar]
trivy image --input Output_file.tar
```
List Image =>
- alpine:3.10
- alpine:latest

4. Filter Vulnerability

- Hide Unfixed Vulnerabilities
```
trivy image nginx
trivy image --ignore-unfixed nginx 
```
- By Severity
```
trivy image --severity HIGH,CRITICAL nginx
trivy image --severity HIGH,CRITICAL --ignore-unfixed nginx
```
- By Vulnerability IDs
```
cat .trivyignore
# Accept the risk
CVE-XXX-YYYY

# No impact in our settings
CVE-XXX-YYYY

trivy image --severity HIGH,CRITICAL --ignore-unfixed nginx
```
- By Type (os, library)

```
trivy image --vuln-type os python:3.4-alpine
trivy image --vuln-type library python:3.4-alpine
```

- debug scan image

```
trivy -d image [Image]
```

Ref: https://aquasecurity.github.io/trivy/v0.22.0/vulnerability/examples/filter/

5. Report Formats
```
# Default table
trivy -d image -f table golang:1.12-alpine

#json
trivy image -f json -o results.json golang:1.12-alpine
```

X. [Tambahan] Scan Git Repository

```
trivy repo https://github.com/alanadiprastyo/demo-devsecops.git
```
Ref: https://aquasecurity.github.io/trivy/v0.22.0/vulnerability/scanning/git-repository/