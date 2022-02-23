## Reconfigure Router

### Check status node
```
[alan@helper okdtest]$ oc get nodes
NAME                                                          STATUS   ROLES           AGE   VERSION
okd-zxxrv-master-0.asia-southeast2-a.c.my-okd-demo.internal   Ready    master          94m   v1.22.3+2cb6068
okd-zxxrv-master-1.asia-southeast2-b.c.my-okd-demo.internal   Ready    master          94m   v1.22.3+2cb6068
okd-zxxrv-master-2.asia-southeast2-c.c.my-okd-demo.internal   Ready    master          94m   v1.22.3+2cb6068
okd-zxxrv-worker-a-2cz2m                                      Ready    worker   79m   v1.22.3+2cb6068
okd-zxxrv-worker-b-vj8jp                                      Ready    worker   78m   v1.22.3+2cb6068
okd-zxxrv-worker-c-zhxcf                                      Ready    worker          55m   v1.22.3+2cb6068
```

### Add label router on worker-a and worker-b
```
[alan@helper okdtest]$ oc edit nodes okd-zxxrv-worker-a-2cz2m
---
apiVersion: v1
kind: Node
metadata:
  labels:
    node-role.kubernetes.io/router: ""
---
```

### Verify node worker
```
[alan@helper okdtest]$ oc get nodes
NAME                                                          STATUS   ROLES           AGE   VERSION
okd-zxxrv-master-0.asia-southeast2-a.c.my-okd-demo.internal   Ready    master          96m   v1.22.3+2cb6068
okd-zxxrv-master-1.asia-southeast2-b.c.my-okd-demo.internal   Ready    master          96m   v1.22.3+2cb6068
okd-zxxrv-master-2.asia-southeast2-c.c.my-okd-demo.internal   Ready    master          96m   v1.22.3+2cb6068
okd-zxxrv-worker-a-2cz2m                                      Ready    router,worker   81m   v1.22.3+2cb6068
okd-zxxrv-worker-b-vj8jp                                      Ready    router,worker   81m   v1.22.3+2cb6068
okd-zxxrv-worker-c-zhxcf                                      Ready    worker          57m   v1.22.3+2cb6068
```
*Note: make sure roles on worker already add "router" labels

### Edit ingress controller
```
oc edit ingresscontroller default -n openshift-ingress-operator
---
  replicas: 2
  nodePlacement:
    nodeSelector:
      matchLabels:
        node-role.kubernetes.io/router: ""
---
```

### Verify the pod router
```
$ oc get pod -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP            NODE                       NOMINATED NODE   READINESS GATES
router-default-7c74cccdbf-gbqkg   1/1     Running   0          82m   10.128.2.15   okd-zxxrv-worker-b-vj8jp   <none>           <none>
router-default-7c74cccdbf-nqznr   1/1     Running   0          82m   10.131.0.26   okd-zxxrv-worker-a-2cz2m   <none>           <none>
```