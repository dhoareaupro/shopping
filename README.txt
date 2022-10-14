Start Docker Images
run  = docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
stop = docker-compose -f docker-compose.yml -f docker-compose.override.yml down
--
See images
docker images

See running containers
docker ps
--
See application locally
TEST
http://localhost:8000/swagger/index.html
http://localhost:8001/
--
Stop Running Containers
stop = docker-compose -f docker-compose.yml -f docker-compose.override.yml down
-- --
Install the Azure CLI
	https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
	cd
--
az version

{
  "azure-cli": "2.16.0",
  "azure-cli-core": "2.16.0",
  "azure-cli-telemetry": "1.0.6",
  "extensions": {}
}
--
az login
--
Create a resource group
az group create --name myResourceGroup --location westeurope
--
Create an Azure Container Registry
az acr create --resource-group myResourceGroup --name shoppingacr --sku Basic
--
Enable Admin Account for ACR Pull
az acr update -n shoppingacr --admin-enabled true
--
Log in to the container registry
az acr login --name shoppingacr
Login Succeeded
--
Tag a container image

get the login server address
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
shoppingacr.azurecr.io
--
Tag your images

docker tag shoppingapi:latest shoppingacr.azurecr.io/shoppingapi:v1
docker tag shoppingclient:latest shoppingacr.azurecr.io/shoppingclient:v1

Check
docker images
--
Push images to registry

docker push shoppingacr.azurecr.io/shoppingapi:v1
docker push shoppingacr.azurecr.io/shoppingclient:v1
--
List images in registry
az acr repository list --name shoppingacr --output table

Result
shoppingapi
shoppingclient
--
See tags
az acr repository show-tags --name shoppingacr --repository shoppingclient --output table

Result
v1
--
Create AKS cluster with attaching ACR
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --generate-ssh-keys --attach-acr shoppingacr

--
Install the Kubernetes CLI
az aks install-cli
--
Connect to cluster using kubectl
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

To verify;
kubectl get nodes
kubectl get all
--
Check Kube Config

kubectl config get-contexts
kubectl config current-context
kubectl config use-context gcpcluster-k8s-1
	Switched to context "gcpcluster-k8s-1"
--
Deploy microservices to AKS

kubectl apply -f .\aks\
--
Clean All AKS and Azure Resources

az group delete --name myResourceGroup --yes --no-wait


83. Run K8s Manifest Yaml Files For Deploying AKS
	kubectl config get-contexts
	kubectl config current-context
	kubectl apply -f .\aks\
Résultat :
configmap/mongo-configmap unchanged
secret/mongo-secret unchanged
deployment.apps/mongo-deployment unchanged
service/mongo-service unchanged
configmap/shoppingapi-configmap unchanged
deployment.apps/shoppingapi-deployment created
service/shoppingapi-service created
horizontalpodautoscaler.autoscaling/shoppingapi-hpa unchanged
horizontalpodautoscaler.autoscaling/shoppingclient-hpa unchanged
deployment.apps/shoppingclient-deployment created
service/shoppingclient-service created

----
Il y a un pb en lancant l'app sur le nvagateur
----
	kubectl get all
	kubectl get svc

kubectl get pod
kubectl describe pod
kubectl describe pod mongo-deployment-6ddfc97dd8-7xv2n

84. Scale Shopping applications in Azure Kubernetes Service (AKS)
	kubectl get deployment
	kubectl get all
		deployment.apps/shoppingclient-deployment 

	kubectl scale --replicas=3 deployment.apps/shoppingclient-deployment
      kubectl get pod --watch
85. Autoscale Shopping Pods in Azure Kubernetes Service (AKS)
	az aks show --resource-group myResourceGroup --name myAKSCluster101222 --query kubernetesVersion --output table
	kubectl get hpa
	kubectl get pod
	kubectl describe pod shoppingclient-deployment-5858d5f7d7-4fzvx
86. Update Shopping Microservices With Zero-Downtime Deployment on Live AKS
	Suppression de l'image shoppingclient via docker rmi 1def65852f17 -f
		suppression de toutes les images

	Il faut recreer via docker-compose une image
	docker-compose -f  .\docker-compose.yml -f .\docker-compose.override.yml up -d
	docker images
	docker ps

	docker-compose -f  .\docker-compose.yml -f .\docker-compose.override.yml down

87. Tag and Push the New Version of Shopping.Client Image to ACR
	az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
	docker images

	docker tag shoppingclient:latest shoppingacr101222.azurecr.io/shoppingclient:v2
	docker images
		ajout d'une images shoppingacr101222.azurecr.io/shoppingclient:v2 avec le tag v2
	
	PUSH image sur le serveur
	docker push shoppingacr101222.azurecr.io/shoppingclient:v2
	Si non authentifié  alors se logger comme ci-dessous
	az acr login --name shoppingacr101222

puis recommencer le push
	docker push shoppingacr101222.azurecr.io/shoppingclient:v2

check la list des repo et ses versions sur azure
	az acr repository show-tags --name shoppingacr101222 --repository shoppingclient --output table
Result
--------
v1
v2
88. Deploy v2 of Shopping.Client Microservices to AKS with zero-downtime rollout k8s
 	kubectl get pod
	kubectl apply -f .\aks\
	kubectl get pod --watch
	kubectl get deployment -o wide
	kubectl get replicaset -o wide
	kubectl describe pod
	kubectl describe pod shoppingclient-deployment-8c9548648

Section 10 : Automate Deployments with CI/CD pipelines on Azure Devops
90. Introduction
91. Introduction to Azure Devops
92. Introduction to Azure Pipelines

93. Deploy to AKS through Azure CI/CD Pipelines
	Step to Azure Devops Deployment
94. Sign up for Azure Pipelines
95. Create Our first pipeline with Azure Pipelines
96. Pipeline Docker Task Yaml File Explained in Azure Pipelines
97. Pipeline Docker Task Error Fixed in Azure Pipelines
	Add this one in the file azurepipeline.yaml
        buildContext: $(build.SourceDirectory)/Shopping
98. Pipeline Tasks and How to Decide to Write Correct Task in Azure Pipelines
99. Create Pipeline for Continues Delivery with Deploy to AKS Task

100. Create Pipeline for Continues Delivery with Deploy to AKS Task Part 2

101. Refactoring Our Pipeline for Continues Delivery with Deploy to AKS
102. Running Pipeline for Continues Delivery with Deploy to AKS
103. Manage Pipelines for Multi-Container Microservices with CI/CD flows in Azure Pip
104. Create New Pipeline For ShoppingAPI Microservices with existing pipeline yaml fi


