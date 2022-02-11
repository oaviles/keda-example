# DevSquad KEDA Project

This repository consists of everything you need to setup simple Kubernetes 
cluster and demonstrate usage of KEDA mysql scalers. For more
samples check https://github.com/kedacore/samples

The included `helper` provides an easy way to perform both 0 -> n and n -> 0 scalings.  

## Create cluster
- You can refer this repro to create ypu AKS Cluster: [DevSquad Cloud-Native Project](https://github.com/oaviles/hello_cloud-native)
- Run GitHub Action called ["Deploy AKS"](https://github.com/oaviles/hello_cloud-native/actions/workflows/deploy-aks.yml)

## Install KEDA
- In this repo, run GitHub Action called ["Setup Keda"](https://github.com/oaviles/keda-example/actions/workflows/setup-keda.yml)
- Optional: Manual deployement: Follow the official KEDA guide https://keda.sh/deploy/


## Deploy KEDA Demo
The deployment consists of 4 components: In this repo, run GitHub Action called ["Deploy Keda Demo"](https://github.com/oaviles/keda-example/actions/workflows/deploy-demo.yml)
- MySQL instance
- Dummy pod that will be scaled up and down
- App service that provides some helper methods



## Observe
To observe how everything works you can watch two things:
- Number of pods and their state: `watch -n2 "kubectl get pods"`
- HPA stats: `watch -n2 "kubectl get hpa"`


## MySQL example
To scale the dummy deployment using 
[MySQL scaler](https://keda.sh/scalers/mysql/) first we have to
deploy the `ScaledObjects`:
```sh
kubectl apply -f keda/mysql-hpa.yaml
```
this should result again in creation of `ScaleObject` and an HPA:
```sh
# kubectl get scaledobjects
NAME                 DEPLOYMENT   TRIGGERS   AGE
mysql-scaledobject   dummy        mysql      5s

# kubectl get hpa
NAME             REFERENCE          TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-dummy   Deployment/dummy   <unknown>/5 (avg)   1         4         0          45s
```

To scale up we have to insert some values to MySQL database. 
To do this we can use the helper app:
```shell script
kubectl exec $(kubectl get pods | grep "server" | cut -f 1 -d " ") -- keda-talk mysql insert
```
and to scale down:
```shell script
kubectl exec $(kubectl get pods | grep "server" | cut -f 1 -d " ") -- keda-talk mysql delete
```
