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

Unzip and cd into dir and upload to GIT
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
