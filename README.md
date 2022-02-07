# Tanzu Application Platform (TAP)

## TAP Install
Write up in progress for the TAP install

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

 Here are a few commands to check things out: 
 
 ```
kubectl get pods -n tap-install --watch

k get routes -n orf
NAME               URL                                          READY   REASON
spring-petclinic   http://spring-petclinic.orf.cnrs.lab.local   True    


k get revisions.serving.knative.dev -n orf
NAME                     CONFIG NAME        K8S SERVICE NAME   GENERATION   READY   REASON   ACTUAL REPLICAS   DESIRED REPLICAS
spring-petclinic-00001   spring-petclinic                      1            True             0                 0

k get gitrepository -A
NAMESPACE            NAME                                 URL                                                                   READY   STATUS                                                               AGE
accelerator-system   hello-fun-acc-2xbkf                  https://github.com/sample-accelerators/hello-fun                      True    Fetched revision: tap-1.0/717160c8a713e1c32e371e4ac788727464f39e89   11d
accelerator-system   hello-ytt-acc-ccq7b                  https://github.com/sample-accelerators/hello-ytt                      True    Fetched revision: tap-1.0/95921a9f9a62a758e11fb4cef612ccb168b89114   11d
accelerator-system   new-accelerator-acc-9qhgz            https://github.com/sample-accelerators/new-accelerator                True    Fetched revision: tap-1.0/b4dc3cf8fd27cef532ab303c2bc8836a53ca4c66   11d
accelerator-system   node-express-acc-rpzhg               https://github.com/sample-accelerators/node-express                   True    Fetched revision: tap-1.0/b0c81e5df690bd2ec0cb6c1bbc0e5e0a32556914   11d
accelerator-system   spring-petclinic-acc-rbpk9           https://github.com/sample-accelerators/spring-petclinic               True    Fetched revision: tap-1.0/334ed57bdda51aba0f671e9ae4b184783b7af74b   11d
accelerator-system   spring-sensors-rabbit-acc-rpmwc      https://github.com/sample-accelerators/spring-sensors                 True    Fetched revision: tap-1.0/cc167e5717122d51bbb2adfd9a52fd4e3a2ffce6   11d
accelerator-system   spring-sql-jpa-acc-mbxg7             https://github.com/sample-accelerators/spring-sql-jpa                 True    Fetched revision: tap-1.0/b3f4c22d0ba1b0d145799a0834a40a2095c490cc   11d
accelerator-system   tanzu-java-web-app-acc-cvsfg         https://github.com/sample-accelerators/tanzu-java-web-app.git         True    Fetched revision: tap-1.0/90bd107ad26e228886e2ad28c964580858196376   11d
accelerator-system   tap-initialize-acc-qkxpq             https://github.com/sample-accelerators/tap-initialize                 True    Fetched revision: tap-1.0/3966b2c5f13e8d6ac593d2a16ce734db29faacad   11d
accelerator-system   weatherforecast-csharp-acc-6ncdd     https://github.com/sample-accelerators/csharp-weatherforecast.git     True    Fetched revision: tap-1.0/81b8aa8288c38c33223e9140359552d527010c0f   11d
accelerator-system   weatherforecast-fsharp-acc-xdnpm     https://github.com/sample-accelerators/fsharp-weatherforecast.git     True    Fetched revision: tap-1.0/a405127123339686825245d455a016ecd59556b1   11d
accelerator-system   weatherforecast-steeltoe-acc-dth6n   https://github.com/sample-accelerators/steeltoe-weatherforecast.git   True    Fetched revision: tap-1.0/4b6a08d43859df65ed598bb309c291f4b9f57569   11d
orf                  spring-petclinic                     https://github.com/ogelbric/spring-petclinic                          True    Fetched revision: main/418aed4cdcfda152b8885dec77bb96e39d5a0831      3h15m

k get images.kpack.io -n orf
NAME               LATESTIMAGE                                                                                                             READY
spring-petclinic   index.docker.io/ogelbric/spring-petclinic-orf@sha256:36cc4776d066fc7f942d56e3ec1326974dc4bb55160a0a5268175ab7d8e93436   True
 
 ```
The repo (docker in this case) will have a new entry for petclinic:

![Version](https://github.com/ogelbric/TAP/blob/main/Docker.png)

