# ensure has access to mcr
# ssh into rover runner
# deploy sample apps
# https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli

vi azure-vote.yaml

# copy and paste content of "/tf/caf/gcc_starter_kit/validation/sample-app-internal-lb.yaml" into azure-vote.yaml

# sandpit tenant - must use this to login
az login --tenant [your tenant id] # htx sandpit - ac20add1-ffda-45c1-adc5-16a0db15810f

# CEP subscription
az account set --subscription [your subscription id] # cep - xxxxxxxx-4066-42f0-xxxxxxxxxxxx

# sample: az aks get-credentials --resource-group escep-rg-aks --name escep-aks
# get aks credentials
az aks get-credentials --resource-group escep-rg-aks-re1 --name escep-aks-cluster-re1

# **IMPORTANT 
# TODO: 
# 1. link the aks private DNS to devops vnet, else will not be able to find AKS private cluster.
# 2. Grant netwrok contributor to gcci-vnet-internet vnet to allow creation of internal load balancer
# virtual network: /subscriptions/xxxxxxxx-4066-42f0-xxxxxxxxxxxx/resourceGroups/gcci-platform/providers/Microsoft.Network/virtualNetworks/gcci-vnet-internet
# permission: Reader and Network Contributor
ERROR:
''' BASH
Error syncing load balancer: failed to ensure load balancer: Retriable: false, RetryAfter: 0s, HTTPStatusCode: 403, RawError: {"error":{"code":"AuthorizationFailed","message":"The client '3448bfd3-5774-4682-a10a-cab46df1e337' with object id '3448bfd3-5774-4682-a10a-cab46df1e337' does not have authorization to perform action 'Microsoft.Network/virtualNetworks/subnets/read' over scope '/subscriptions/xxxxxxxx-4066-42f0-xxxxxxxxxxxx/resourceGroups/gcci-platform/providers/Microsoft.Network/virtualNetworks/gcci-vnet-internet/subnets/escep-snet-app-internet' or the scope is invalid. If access was recently granted, please refresh your credentials."}}
'''
Resolution:
Grant "reader" and "network contributor" to managed identity "escep-msi-aks-usermsi" (object id: 3448bfd3-5774-4682-a10a-cab46df1e337) to vnet "escep-snet-app-internet"

Reference: https://learn.microsoft.com/en-us/answers/questions/720289/i-am-unable-to-spin-up-internal-load-balancers-in
# Note the load balancer will take about 2 minutes to create.


# deploy the container into aks
kubectl apply -f azure-vote.yaml

# remove the container into aks
kubectl delete -f azure-vote.yaml

Issue:
1. Ensure age port 80 is the listener (private) if http or 443 is using private listener if https
