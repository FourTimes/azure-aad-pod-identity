Azure Active Directory Pod Identity provides token-based access to Azure Resource Manager (ARM).


AAD Pod Identity will add the following components to your Kubernetes cluster: 
    
    1. Kubernetes CRDs: 
        AzureIdentity
        AzureAssignedIdentity
        AzureIdentityBinding 
    2. Managed Identity Controller (MIC) component 
    3. Node Managed Identity (NMI) component

To install AAD Pod Identity to your cluster:

    

```bash

# RBAC enabled AKS cluster

kubectl apply -f https://raw.githubusercontent.com/Azure/aad-pod-identity/v1.6.0/deploy/infra/deployment-rbac.yaml

```
```bash

#RBAC disabled AKS cluster
    
kubectl apply -f https://raw.githubusercontent.com/Azure/aad-pod-identity/v1.6.0/deploy/infra/deployment.yaml

```

Install Helm 

    Helm is a package manager for Kubernetes.

```bash

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
bash get_helm.sh

```

```bash

# Make sure aks credentials should be download
helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
helm repo update

# Find the latest version of application-gateway-kubernetes-ingress
helm search repo -l application-gateway-kubernetes-ingress

```

Prerequisites

    1. Application gateway
    2. Managed Identity


Download helm-config.yaml, which will configure AGIC: 

```bash

wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/sample-helm-config.yaml -O helm-config.yaml

```


```bash

# vim helm-config.yaml

# Verbosity level of the App Gateway Ingress Controller
verbosityLevel: 3

# application gateway details
appgw:
    subscriptionId: a0808583-341e-4259-a7f8-10331546798f
    resourceGroup: xxxx-portal-dev
    name: dev-aks
    usePrivateIP: false

    # Setting appgw.shared to "true" will create an AzureIngressProhibitedTarget CRD.
    # This prohibits AGIC from applying config for any host/path.
    # Use "kubectl get AzureIngressProhibitedTargets" to view and change this.
    shared: false

# Specify which kubernetes namespace the ingress controller will watch
# Default value is "default"
# Leaving this variable out or setting it to blank or empty string would
# result in Ingress Controller observing all accessible namespaces.
#
# kubernetes:
#   watchNamespace: <namespace>

# Specify the authentication with Azure Resource Manager. Two authentication methods are available:
# - Option 1: AAD-Pod-Identity (https://github.com/Azure/aad-pod-identity)
armAuth:
    type: aadPodIdentity
    identityResourceID: /subscriptions/xxxxxx-xxxx-xxx-a7f8-xxxxxx/resourceGroups/xxxx-portal-dev/providers/Microsoft.ManagedIdentity/userAssignedIdentities/apg-keyvault
    identityClientID:  xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx

## Alternatively you can use Service Principal credentials
# armAuth:
#    type: servicePrincipal
#    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --subscription <subscription-uuid> --sdk-auth | base64 -w0" >>

# Specify if the cluster is RBAC enabled or not
rbac:
    enabled: false # true/false


```