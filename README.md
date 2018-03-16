# Setting up jenkins in AKS

## Create the AKS cluster and initial helm 

```
az aks create -n $CLUSTER_NAME -g $RESOURCE_GROUP
az aks get-credentials -n mycluster -g myresourcegroup -l $LOCATION
helm init 
```

## Setup a persistence volume claim for Jenkins 

### Create the storage account 

> Make sure you use the resource group of the AKS nodes which is usually of the form ` MC_"$RESOURCE_GROUP"_"$AKS_CLUSTER_NAME"_"$LOCATION"`

```
az storage account create -n $STORAGE_ACCOUNT -g $RESOURCE_GROUP_AKS -l $LOCATION --sku Standard_LRS
```

### Create the storage class

Edit [azure-file-sc.yaml](azure-file.sc.yaml) and replace with the storage account created. 

```
kubectl create -f azure-file-sc.yaml
```

Alternatively use envsubst and run the following command 

```
envsubst < azure-file-sc.yaml | kubectl create -f -
```

### Create Persistent Volume Claim

```
kubectl create -f azure-file-pvc.yaml
```

## Install Jenkins using the persistence claim 

```
helm install --name jenkins-release-1 --set Persistence.ExistingClaim=$PVC_NAME stable/jenkins
```
