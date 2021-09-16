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

```

```bash
# Find the latest version of application-gateway-kubernetes-ingress
helm search repo -l application-gateway-kubernetes-ingress


NAME                                              	CHART VERSION	APP VERSION	DESCRIPTION                                       
application-gateway-kubernetes-ingress/ingress-...	1.4.0        	1.4.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	1.3.0        	1.3.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	1.2.1        	1.2.1      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	1.2.0        	1.2.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	1.0.0        	1.0.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.10.0       	0.10.0     	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.9.0        	0.9.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.8.0        	0.8.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.7.1        	0.7.1      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.7.0        	0.7.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.6.0        	0.6.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.5.0        	0.5.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.4.0        	0.4.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.3.0        	0.3.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.2.0        	0.2.0      	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.1.5        	1.0        	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.1.4        	1.0        	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.1.3        	1.0        	Use Azure Application Gateway as the ingress fo...
application-gateway-kubernetes-ingress/ingress-...	0.1.2        	1.0        	Use Azure Application Gateway as the ingress fo...

```

Prerequisites

    1. Application gateway
    2. Managed Identity


Require Parameters:

    1. resourceGroup
    2. subscriptionId
    3. applicationGatewayName
    4. identityResourceID
    5. identityClientID

Download helm-config.yaml, which will configure AGIC: 

```bash

wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/sample-helm-config.yaml -O helm-config.yaml

```


```bash

# vim helm-config.yaml

image:
  repository: mcr.microsoft.com/azure-application-gateway/kubernetes-ingress
  tag: 1.4.0
  
# Verbosity level of the App Gateway Ingress Controller
verbosityLevel: 3

# application gateway details
appgw:
    subscriptionId: xxxxxx-xxxx-xxx-a7f8-xxxxxx
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

Installation of AGIG

```bash

    # Installtion of ingress-azure
    helm install ingress-azure  -f helm-config.yaml  application-gateway-kubernetes-ingress/ingress-azure --version 1.4.0

```

output:

```bash

NAME: ingress-azure
LAST DEPLOYED: Thu Sep  2 05:11:40 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing ingress-azure:1.4.0.

Your release is named ingress-azure.
The controller is deployed in deployment ingress-azure.

Configuration Details:
----------------------
 * AzureRM Authentication Method:
    - Use AAD-Pod-Identity
 * Application Gateway:
    - Subscription ID : xxxxxx-xxxx-xxx-a7f8-xxxxxx
    - Resource Group  : xxxx-portal-dev
    - Application Gateway Name : dev-aks
 * Kubernetes Ingress Controller:
    - Watching All Namespaces
    - Verbosity level: 3

Please make sure the associated aadpodidentity and aadpodidbinding is configured.
For more information on AAD-Pod-Identity, please visit https://github.com/Azure/aad-pod-identity

```

Pods running status

```sh
NAME                             READY   STATUS    RESTARTS   AGE
ingress-azure-5ff4f9bbd9-dxvqp   1/1     Running   0          3h9m
mic-5bfcb9b7cd-5jzsx             1/1     Running   0          3h15m
mic-5bfcb9b7cd-rr6hp             1/1     Running   0          3h15m
nmi-44hf8                        1/1     Running   0          3h15m


```


Demo Application Deployment

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: dotnet
  labels:
    app: dotnet
spec:
  containers:
  - image: jjino/dotnet
    name: dotnet
    ports:
    - containerPort: 80
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: dotnet
  labels:
    app: dotnet
spec:
  selector:
    app: dotnet
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dotnet
  labels:
    app: dotnet
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: dotnet
          servicePort: 80
EOF

```

Results:

```sh

# kubectl get po,svc,ingress -l app=dotnet

NAME                        READY       STATUS      RESTARTS        AGE
pod/dotnet                  1/1         Running     0               3h19m

NAME                        TYPE        CLUSTER-IP  EXTERNAL-IP     PORT(S)     AGE
service/dotnet              ClusterIP   10.2.7.203  <none>          80/TCP      3h19m

NAME                        CLASS       HOSTS       ADDRESS         PORTS       AGE
ingress.extensions/dotnet   <none>      *           40.88.217.157   80          3h19m

```
