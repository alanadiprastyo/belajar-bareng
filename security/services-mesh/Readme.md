# Install Istio
-----

Install Getmesh
```
curl -sL https://istio.tetratelabs.io/getmesh/install.sh | bash
```

install istio
```
getmesh istioctl install --set profile=demo
```
output
```
✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
  To get started, check out https://istio.io/latest/docs/setup/getting-started/

This will install the Istio 1.13.3 demo profile with ["Istio core" "Istiod" "Ingress gateways" "Egress gateways"] components into the cluster. Proceed? (y/N) y
✔ Istio core installed                                                                                                                                                      
✔ Istiod installed                                                                                                                                                          
✔ Ingress gateways installed                                                                                                                                                
✔ Egress gateways installed                                                                                                                                                 
✔ Installation complete                                                                                                                                                     Making this installation the default for injection and validation.
```

Check version
```
getmesh version
```
output
```
getmesh version: 1.1.4
active istioctl: 1.13.3-tetrate-v0
no running Istio pods in "istio-system"
1.13.3-tetrate-v0
```

check validate getmesh
```
getmesh config-validate 
```
show getmesh
```
getmesh show
```
list getmesh
```
getmesh list
```