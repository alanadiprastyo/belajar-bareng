### Setting Dynamic Storage Class with NFS Server

1. Setting NFS Server
```
mkdir /shares/nfs-services
chown nobody:nobody /shares/nfs-services/ -R
chmod -R 777 /shares/nfs-services/

vi /etc/exports
/shares/nfs-services 192.168.22.0/24(rw,sync,root_squash,no_subtree_check,no_wdelay)

systemctl restart nfs-server rpcbind
```

2. Setting NFS Provisioner on OCP
```
git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

oc create ns openshift-nfs-storage
oc label namespace openshift-nfs-storage "openshift.io/cluster-monitoring=true"

cd nfs-subdir-external-provisioner/deploy
oc create -f rbac.yaml
oc adm policy add-scc-to-user hostmount-anyuid system:serviceaccount:openshift-nfs-storage:nfs-client-provisioner
```

3. apply storage class
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: helper-nfs-storage
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
```
```
oc apply -f class.yaml
```

4. apply deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: openshift-nfs-storage
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.22.1
            - name: NFS_PATH
              value: /shares/nfs-services
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.22.1
            path: /shares/nfs-services
```
```
oc apply -f deployment.yaml
```

5. verify storage class
```
oc get sc

NAME                 PROVISIONER                                   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
helper-nfs-storage   k8s-sigs.io/nfs-subdir-external-provisioner   Delete          Immediate           false                  14m
```

6. Test create pv
```
oc apply -f test-claim.yaml
oc apply -f test-pod.yaml
```