# K8S
## Deploying Kubernetes Dashboard.

1.  The Dashboard UI is not deployed by default. To deploy it, run the following command:

Copy
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

## NodePort

This way of accessing Dashboard is only recommended for development environments in a single node setup.

Edit kubernetes-dashboard service.

```$ kubectl -n kube-system edit service kubernetes-dashboard```

You should see yaml representation of the service. Change type: ClusterIP to type: NodePort and save file. If it's already changed go to next step.

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
...
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "343478"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard-head
  uid: 8e48f478-993d-11e7-87e0-901b0e532516
spec:
  clusterIP: 10.100.124.90
  externalTrafficPolicy: Cluster
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

Verify that the K8S dashboard is exposed on NodePort.

```
~]# kubectl get service -n kube-system kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.109.238.150   <none>        443:31038/TCP   146m
```

You can access the K8S dashboard from your browser at 
```
https://<node-ip>:<nodePort>
```

## 1. Create User
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```
## 2. Bind user to CreateRole
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

## 3. Create a Bearer Token
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

  
