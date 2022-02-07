# Tanzu Application Platform (TAP)

## TAP Install

## TAP Application install (petclinc)

Select in TAP GUI an application and locate zip file:

![Version](https://github.com/ogelbric/TAP/blob/main/Petclinicdownload.png)

Log onto Tanzu guest cluster and delete old deployments in user namespace(if there are any / my case ns=orf)
```
kubectl vsphere login --server 192.168.5.41 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace1000 --tanzu-kubernetes-cluster-name workload-vsphere-tkg1 --insecure-skip-tls-verify

tanzu apps workload delete weatherforecast -n orf
```

Unzip and cd into dir and upload to Git
```
unzip spring-petclinic.zip
cd spring-petclinic
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/ogelbric/spring-petclinic.git
curl -u ogelbrich:ghp_VxxxxxxxmGaxxxxMyKEYXxxxxxxxxxxxxxxxx https://api.github.com/user/repos -d '{"name":"spring-petclinic"}'
git push -u origin main
Login with ID and key as password (key is obtained in the profile section in GIT)
```
Result in Git should look like this:

![Version](https://github.com/ogelbric/TAP/blob/main/GitResult.png)

Create a namespace and docker secret (or other registry) for application (kubectl is aliased to k) (notice for diocker /v1/ is a must!):
```
k create ns orf

tanzu secret registry add registry-credentials --server https://index.docker.io/v1/ --username ogelbric --password 'xxxxxxxx!' --namespace orf

```

Create this in the namespace (in my case ns=orf)

```
cat <<EOF | kubectl -n orf apply -f -

apiVersion: v1
kind: Secret
metadata:
  name: tap-registry
  annotations:
    secretgen.carvel.dev/image-pull-secret: ""
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: e30K

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
secrets:
  - name: registry-credentials
imagePullSecrets:
  - name: registry-credentials
  - name: tap-registry

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: default
rules:
- apiGroups: [source.toolkit.fluxcd.io]
  resources: [gitrepositories]
  verbs: ['*']
- apiGroups: [source.apps.tanzu.vmware.com]
  resources: [imagerepositories]
  verbs: ['*']
- apiGroups: [carto.run]
  resources: [deliverables, runnables]
  verbs: ['*']
- apiGroups: [kpack.io]
  resources: [images]
  verbs: ['*']
- apiGroups: [conventions.apps.tanzu.vmware.com]
  resources: [podintents]
  verbs: ['*']
- apiGroups: [""]
  resources: ['configmaps']
  verbs: ['*']
- apiGroups: [""]
  resources: ['pods']
  verbs: ['list']
- apiGroups: [tekton.dev]
  resources: [taskruns, pipelineruns]
  verbs: ['*']
- apiGroups: [tekton.dev]
  resources: [pipelines]
  verbs: ['list']
- apiGroups: [kappctrl.k14s.io]
  resources: [apps]
  verbs: ['*']
- apiGroups: [serving.knative.dev]
  resources: ['services']
  verbs: ['*']
- apiGroups: [servicebinding.io]
  resources: ['servicebindings']
  verbs: ['*']
- apiGroups: [services.apps.tanzu.vmware.com]
  resources: ['resourceclaims']
  verbs: ['*']
- apiGroups: [scanning.apps.tanzu.vmware.com]
  resources: ['imagescans', 'sourcescans']
  verbs: ['*']

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: default
subjects:
  - kind: ServiceAccount
    name: default

EOF
```

Outcome:
```
secret/tap-registry configured
serviceaccount/default configured
role.rbac.authorization.k8s.io/default unchanged
rolebinding.rbac.authorization.k8s.io/default unchanged
```

Run this for the application (the names have to match up): 

```
tanzu apps workload create spring-petclinic \
--git-repo https://github.com/ogelbric/spring-petclinic \
--git-branch main \
--type web \
--app spring-petclinic \
--yes --namespace orf
```

![Version](https://github.com/ogelbric/TAP/blob/main/TAPoutcome1.png)

The tap-install.yaml file has the following entry:

```
cnrs:
  domain_name: cnrs.lab.local
```
 
 This command will add it to an existing TAP deploy: 
 
```
 tanzu package installed update tap -p tap.tanzu.vmware.com -v 1.0.0 --values-file tap-values.yaml -n tap-install
```

 DNS server has a star record for the cnrs sub domain which points to the ingress:
 
 ```
 [root@orfdns tap]# k get svc -n tanzu-system-ingress
NAME      TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
contour   ClusterIP      198.51.100.181   <none>         8001/TCP                     11d
envoy     LoadBalancer   198.51.100.113   192.168.5.45   80:32060/TCP,443:31124/TCP   11d
[root@orfdns tap]# 
```

![Version](https://github.com/ogelbric/TAP/blob/main/DNS.png)

 
