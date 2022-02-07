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

