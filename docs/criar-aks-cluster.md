![Portal Azure](https://dytvr9ot2sszz.cloudfront.net/wp-content/uploads/2020/05/k8saks1-1.jpg)
# Criando um Cluster kubernete no AKS
Esse tutorial tem como objetivo ajudar a criar um cluster kubernete no Azure AKS. Eu considero que você já tenha uma conta no Azure e que já tenha instalado e configurado o Azure CLI. Se você não tem uma conta no Azure, crie uma [aqui](https://azure.microsoft.com/pt-br/free/). Se você não tem o Azure CLI instalado, instale [aqui](https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli-linux?pivots=apt).  Testei ele usando Ubuntu 22.04.1 LTS porém você pode utilizá-lo no Windows via WSL.

![Badge](https://img.shields.io/badge/Status-em%20desenvolvimento-yellow) 
[![Azure](https://img.shields.io/badge/Azure-Portal-darkblue)](https://portal.azure.com/) [![AKS](https://img.shields.io/badge/AKS-kubernete-darkblue)](https://learn.microsoft.com/en-us/azure/aks/) [![kubernete](https://img.shields.io/badge/kubernete-Cluster-darkblue)](https://kubernete.io/docs/concepts/architecture/) [![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04+-orange)](http://ubuntu.com) [![Windows](https://img.shields.io/badge/Windows-WSL-blue)](https://learn.microsoft.com/pt-br/windows/wsl/install)
[![made-with-VSCode](https://img.shields.io/badge/Made%20With-VSCode-1f425f.svg)](https://code.visualstudio.com/) 
[![GPLv3 license](https://img.shields.io/badge/License-GPLv3-blue.svg)](http://perso.crans.org/besson/LICENSE.html)

## Exportar variáveis de ambiente
Crie variáveis de ambiente para facilitar a criação do cluster.
```bash
AKS_NAME="nome do cluster"
AKS_RG="nome para o resource group"
SUBSCRIPTION="nome da sua assinatura"
REGION="região do cluster"
AKS_SP="nome do service"
NODE_COUNT=1
NODE_VM_SIZE="Standard_B2s"
```
## Criar um Resource Group
Separar os recursos por grupos facilita a administração e a organização. 
```bash
az group create \
  --location $REGION \
  --name $AKS_RG \
  --subscription $SUBSCRIPTION
```
## Criar o Service of Principle
Crie um Service of Principle para o seu cluster. 
```bash
  az ad sp create-for-rbac \
  --skip-assignment \
  -n $AKS_SP
```
## Criar o Cluster
Crie um cluster usando o id e senha do Service criado anteriormente. Nesse exemplo criei com as tags ENV e SRV. Importante criar o cluster com tags para facilitar a organização e a administração.
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
  --node-vm-size $NODE_VM_SIZE \
  --node-count $NODE_COUNT \
  --tags 'ENV=DEV' 'SRV=EXAMPLE'
```
## Gerar o arquivo de configuração do kubeconfig
Crie o diretório .kube caso ele não exista.
```bash
mkdir ~/.kube
```
Crie a variável FILE com o caminho do arquivo de configuração do kubeconfig.
```bash	
FILE="$HOME/.kube/$AKS_NAME.kubeconfig"
```
Gere o arquivo de configuração do kubeconfig com as credenciais do cluster.
```bash	
az aks get-credentials \
  --name $AKS_NAME \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --admin \
  --file $FILE
```
Exporte a variável de ambiente KUBECONFIG com o caminho do arquivo de configuração do kubeconfig.
```bash
export KUBECONFIG=$FILE
```
## Verificar o cluster criado
Verifique se o cluster criado. 
```bash
kubectl get nodes
```
## Criar um pod do tipo busybox para testar o cluster
Crie um pod do tipo busybox para testar o cluster. Isso é muito útil para testar o cluster.
```bash
kubectl run -it --rm busybox --image=busybox -- sh
```