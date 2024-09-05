# Expondo Múltiplas Aplicações no Amazon EKS com um Único Application Load Balancer

Este repositório contém um guia para expor várias aplicações em um cluster do Amazon EKS utilizando um único Application Load Balancer (ALB). O objetivo é demonstrar como implementar uma arquitetura de microsserviços de forma simples e econômica.

## Pré-requisitos

Antes de começar, você precisará ter:

- Um cluster do Amazon EKS provisionado com Node Groups ou um Fargate Profile.
- O AWS Load Balancer Controller configurado no seu cluster.
- Imagens de container das suas aplicações disponíveis em um repositório do Amazon Elastic Container Registry (ECR) ou outro repositório de sua escolha.

## Estrutura do Projeto

O projeto inclui:

- **Aplicações de exemplo**: Duas aplicações simples que exibem páginas HTML com fundo verde e amarelo.
- **Dockerfiles**: Arquivos de configuração para construir as imagens Docker.
- **Arquivo de configuração do Kubernetes**: Um arquivo YAML que define os recursos Kubernetes necessários.

## Passo a Passo

### 1. Criação das Aplicações e Imagens Docker

1. Crie dois diretórios para as aplicações:
   ```bash
   mkdir green yellow
   ```

2. Crie os arquivos HTML para cada aplicação:
   - **Aplicação Verde**:
     ```bash
     echo '<html style="background-color: green;"></html>' > green/index.html
     ```
   - **Aplicação Amarela**:
     ```bash
     echo '<html style="background-color: yellow;"></html>' > yellow/index.html
     ```

3. Crie os Dockerfiles:
   - **Dockerfile da Aplicação Verde**:
     ```Dockerfile
     FROM nginx:alpine
     RUN mkdir -p /usr/share/nginx/html/green
     COPY ./index.html /usr/share/nginx/html/green/index.html
     EXPOSE 80
     ```
   - **Dockerfile da Aplicação Amarela**:
     ```Dockerfile
     FROM nginx:alpine
     RUN mkdir -p /usr/share/nginx/html/yellow
     COPY ./index.html /usr/share/nginx/html/yellow/index.html
     EXPOSE 80
     ```

4. Crie os repositórios no Amazon ECR e envie as imagens:
   ```bash
   aws ecr create-repository --repository-name labs-green-app
   aws ecr create-repository --repository-name labs-yellow-app
   # Siga as instruções de push fornecidas pelo ECR
   ```

5. Adicione as variáveis de ambiente:
```bash
export AWS_REGION=$(aws ec2 describe-availability-zones --output text --query 'AvailabilityZones[0].[RegionName]')
export AWS_REGISTRY_ID=$(aws ecr describe-registry --query registryId --output text)
export AWS_ECR_REPO=${AWS_REGISTRY_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
```
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWS_ECR_REPO

```

### 2. Configuração do Ambiente no Kubernetes

1. O arquivo `color-app.yaml` é a definição e configuração do serviço

### 3. Implementação no Cluster EKS

1. Define o cluster EKS que irá criar o deployment:

```bash
aws eks update-kubeconfig --name my-eks-cluster --region {region-name}
```

2. Acesse o diretório onde o `color-app.yaml` foi salvo e execute:
   ```bash
   kubectl apply -f color-app.yaml
   ```

3. Após o provisionamento do ALB, obtenha a URL de acesso:
   ```bash
   kubectl get ingress labs-color-app-api-ingress -n labs-color-app-api -o=jsonpath="{'http://'}{.status.loadBalancer.ingress[].hostname}{'\n'}"
   ```

## Conclusão

Neste guia, abordamos como expor múltiplas aplicações em um cluster do Amazon EKS utilizando um único Application Load Balancer. Essa abordagem permite um gerenciamento eficiente e uma redução de custos ao operar aplicações na nuvem.

## Referência

- Rubens Devito: Arquiteto de Soluções Sênior na AWS.
- JP Santana: Arquiteto de Soluções Principal para Startups na AWS.
- [Documentação](https://aws.amazon.com/pt/blogs/aws-brasil/como-expor-multiplas-aplicacoes-no-amazon-eks-utilizando-um-unico-application-load-balancer/) 

## Licença

Este projeto está licenciado sob a [Licença MIT](LICENSE).

## Comandos Essencias

Comandos utilizados para debugar ou corrigir problemas durante o processo

```bash
# Listar pods e seus status
kubectl get pods -n labs-color-app-api

#Describe de um pod especifico
kubectl describe pod labs-yellow-app-85cf8467b4-4ntjp -n labs-color-app-api

#Sobrescrever deployment
kubectl replace -f color-app.yaml

# Deletar todos os pods que iniciam com yellow-app
kubectl delete all --selector=app=yellow-app -n labs-color-app-api

#Resize de nodes do cluster
eksctl scale nodegroup --cluster my-eks-cluster --name ganso --nodes 2
 
```