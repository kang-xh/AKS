# Setup environment
Jenkins: http://kangxhpipseaoss.southeastasia.cloudapp.azure.com:8080
Grafana: http://kangxhpipseaoss.southeastasia.cloudapp.azure.com:3000

#pwd: dX2iSKg0ynjFQ  
#spn: CBTTGbfLtswbZvIAfe5Lk/YEPwpGILxUToNkVjdDI5w=

### Monitor

prometheus-alertmanager         ClusterIP   10.0.87.231
prometheus-pushgateway          ClusterIP   10.0.207.178
prometheus-server               ClusterIP   10.0.68.176

kubectl --namespace monitoring port-forward prometheus-server-8666645ff5-cgm6z 9090
az aks browse --resource-group az-rg-kangxh-aks --name kangxhakssea

### Manage ACR from kangxhvmseaoss
    az login
    az acr login --name kangxhacrsea
    az acr list --resource-group az-rg-kangxh-aks --query "[].{acrLoginServer:loginServer}" --output table
    az acr repository list --name kangxhacrsea --output table

    az acr repository show-tags --name kangxhacrsea --repository vote-web --output table
    az acr repository delete --name kangxhacrsea --repository vote-web

### vote

    docker image build /home/allenk/github/PoC-AKS/vote/web -t kangxhacrsea.azurecr.io/vote-web:latest
    docker push kangxhacrsea.azurecr.io/vote-web:latest

    docker image build /home/allenk/github/PoC-AKS/vote/api -t kangxhacrsea.azurecr.io/vote-api:latest
    docker push kangxhacrsea.azurecr.io/vote-api:latest

    kubectl apply -f /home/allenk/github/PoC-AKS/vote/vote-deploy.yaml
    kubectl get svc -n vote

    kubectl exec -it vote-api-xxxx -n vote -- /bin/bash

### Tasks

    Tasks Container is a simple API app, Python, Flask. Using in memory list as DB and export service via NodePort for internal access.

    docker image build /home/allenk/github/PoC-AKS/tasks -t kangxhacrsea.azurecr.io/task-api:latest
    docker push kangxhacrsea.azurecr.io/task-api:latest

    kubectl apply -f /home/allenk/github/PoC-AKS/tasks/task-deploy.yaml

    GET
    curl http://10.10.0.4:30030/api/tasks
    curl http://10.10.0.4:30030/api/tasks/1

    POST
    curl -i -H "Content-Type: application/json" -X POST -d '{"title":"Read a book"}' http://10.0.0.4:30030/api/tasks

# namespace
kubectl apply -f ./other/namespace.yaml

# secrets
kubectl apply -f ./secrets/azure-cosmosdb-election-secret.yaml
kubectl apply -f ./secrets/azure-cosmosdb-candidate-secret.yaml
kubectl apply -f ./secrets/azure-cosmosdb-voter-secret.yaml
kubectl apply -f ./secrets/azure-service-bus-secret.yaml

# verify secrets data
kubectl get secret -n voter-api 

# create Pod with Deployment
kubectl apply -f ./services/election-deployment.yaml
kubectl apply -f ./services/candidate-deployment.yaml
kubectl apply -f /services/voter-deployment.yaml

# verify deployment and pod
kubectl get deployment -n voter-api -o wide
kubectl get pod -n voter-api -o wide









