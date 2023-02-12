![Portal Azure](https://devopstales.github.io/img/azure.webp)
# Criando um Cluster AKS
Esse tutorial tem como objetivo criar um cluster AKS no Azure. Eu considero que você já tenha uma conta no Azure e que já tenha instalado e configurado o Azure CLI. Se você não tem uma conta no Azure, crie uma [aqui](https://azure.microsoft.com/pt-br/free/). Se você não tem o Azure CLI instalado, instale [aqui](https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli-linux?pivots=apt).  Testei ele usando Ubuntu 22.04.1 LTS porém você pode utilizá-lo no Windows via WSL.

![Badge](https://img.shields.io/badge/Status-Em%20desenvolvimento-yellow) 
![Azure](https://img.shields.io/badge/Azure-Cloud-darkblue) ![AKS](https://img.shields.io/badge/AKS-Kubernetes-darkblue) ![Kubernetes](https://img.shields.io/badge/Kubernetes-Cluster-darkblue) 
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04+-orange) ![Windows](https://img.shields.io/badge/Windows10-WSL-blue) ![Windows](https://img.shields.io/badge/Windows11-WSL-blue) 
![License](https://img.shields.io/badge/License-BSD-red)

## Exportar variaveis de ambiente
Utilize variaveis de ambiente para facilitar a criação do cluster AKS. 
```bash
AKS_NAME="nome do cluster"
AKS_RG="nome para o resource group"
SUBSCRIPTION="nome da sua assinatura"
REGION="região do cluster"
AKS_SP="nome do service principal"
```

## Criar um Resource Group
Separar os recursos por grupos facilita a administração e a organização. 
```bash
az group create \
  --location $REGION \
  --name $AKS_RG \
  --subscription $SUBSCRIPTION
```

## Criar um Service Principal no AD
Criar um Service Principal no AD para o cluster AKS. 
```bash
  az ad sp create-for-rbac \
  --skip-assignment \
  -n $AKS_SP
```

## Criar um Cluster AKS
Criar um cluster AKS com o Service Principal criado anteriormente. Nesse exemplo criei com as tags ENV e SRV. Importante criar o cluster com tags para facilitar a organização e a administração.
```bash
  az aks create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "substituir pelo id gerado no comando para criar o service" \
  --client-secret "substituir pelo id gerado no comando para criar o service" \
  --network-plugin kubenet \
  --load-balancer-sku basic \
  --outbound-type loadBalancer \
  --node-vm-size Standard_B2s \
  --node-count 1 \
  --tags 'ENV=DEV' 'SRV=EXAMPLE'
```

## Gerar o arquivo de configuração do kubeconfig
Gerar o arquivo de configuração do kubeconfig para o cluster AKS para podermos acessar o cluter criado. 
Criar o diretório .kube caso ele não exista.
```bash
mkdir ~/.kube
```
Criar a variavel FILE com o caminho do arquivo de configuração do kubeconfig.
```bash	
FILE="$HOME/.kube/$AKS_NAME.kubeconfig"
```
Gerar o arquivo de configuração do kubeconfig com as credenciais do cluster.
```bash	
az aks get-credentials \
  --name $AKS_NAME \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --admin \
  --file $FILE
```
Exportar a variavel de ambiente KUBECONFIG com o caminho do arquivo de configuração do kubeconfig.
```bash
export KUBECONFIG=$FILE
```

## Verificar o cluster criado
Verificar o cluster criado. 
```bash
kubectl get nodes
```

## Criar um pod do tipo busybox para testar o cluster
Criar um pod do tipo busybox para testar o cluster. Esse comando vai criar um pod do tipo busybox e vai abrir um terminal para podermos executar comandos dentro do pod. Isso é muito útil para testar o cluster.
```bash
kubectl run -it --rm busybox --image=busybox -- sh
```