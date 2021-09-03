```Dockerfile

FROM mcr.microsoft.com/azure-application-gateway/kubernetes-ingress:1.4.0
USER root
RUN apt update && apt upgrade -y

```
