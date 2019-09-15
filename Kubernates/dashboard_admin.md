https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

### Delpoing dashboard 
  ```sh
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
  ```
Create Service Account

Copy provided snippets to some dashboard-adminuser.yaml file and use 

```sh kubectl apply -f dashboard-adminuser.yaml ```to create them.


We are creating Service Account with name admin-user in namespace kube-system first.

```sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```


###Bearer Token
Now we need to find token we can use to log in. Execute following command:
```sh
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user
```


### Admin privileges
IMPORTANT: Make sure that you know what you are doing before proceeding. Granting admin privileges to Dashboard's Service Account might be a security risk.

You can grant full admin privileges to Dashboard's Service Account by creating below ClusterRoleBinding. Copy the YAML file based on chosen installation method and save as, i.e. dashboard-admin.yaml. Use kubectl create -f dashboard-admin.yaml to deploy it. Afterwards you can use Skip option on login page to access Dashboard.

Official release
  ```sh
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
  ```