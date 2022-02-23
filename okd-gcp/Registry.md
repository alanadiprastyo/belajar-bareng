# Reconfigure Image Registry

### Enable Image Registry Default Route
```
oc patch configs.imageregistry.operator.openshift.io/cluster --type merge -p '{"spec":{"defaultRoute":true}}'
```

### Verify the Default Route Registry
```
$ oc get route
NAME            HOST/PORT                                                              PATH   SERVICES         PORT    TERMINATION   WILDCARD
default-route   default-route-openshift-image-registry.apps.okd.cloudnusantara.my.id          image-registry   <all>   reencrypt     None
```