# Documentação Técnica: Projeto DevSecOps com CI/CD e GitActions

Projeto: GitOps na prática

Versão: 1.0

Data: 25 de Setembro de 2025


## Sumário



------------------

## 1. Visão Geral

Esta documentação é para a construção de um pipeline de CI/CD, começando com "Hello World" em uma aplicação FastAPI que é containerizada via GitHub Actions e enviada ao Docker Hub. Utilizando os princípios do GitOps, o Argo CD assume a responsabilidade pela entrega contínua, implantando a aplicação em um cluster Kubernetes de forma declarativa e segura. 

### 1.1. Tecnologias Utilizadas:

[GitHub](https://github.com/): Plataforma de hospedagem do repositório Git, servindo como a fonte da configuração da aplicação.

[Rancher-Desktop](https://rancherdesktop.io/): Containerização e orquestração dos serviços.

[Kubernetes](https://kubernetes.io/releases/download/): Plataforma de orquestração de containers, responsável por gerenciar o ciclo de vida da aplicação de microserviços.

[ArgoCD](https://github.com/badtuxx/DescomplicandoArgoCD/blob/main/pt/src/day-1/README.md#instalando-o-argocd): O link do ArgoCD irá encaminhar para o repositório do [badtuxx](https://github.com/badtuxx/). Siga as instruções de instalação dele. O ArgoCD é uma ferramenta de GitOps para Kubernetes, utilizada para automatizar o deploy e a sincronização da aplicação a partir do repositório Git.

------------------

## 2. Arquitetura da Solução

Um código é enviado para o GitHub e após isto, o GitHub Actions envia o container que será criado para o Docker Hub.

Continuous Integration(CI): O GitHub Actions detecta o envio, constrói uma imagem Docker da aplicação e a publica no Docker Hub com uma tag de versão única.

Continuous Delivery (CD): O Argo CD, que monitora constantemente o repositório Git, percebe a atualização da tag da imagem nos arquivos de manifesto e automaticamente sincroniza essa nova versão com o cluster Kubernetes, atualizando a aplicação em produção.

[CI e GitHub Actions/](https://github.com/josewolffsantiago/-CompassUOL-CI-CD-com-o-Github-Actions-Sprint6-PBJUN2025-DevSecOPs)
            
            ├── .github/
            │   └── workflows/
            │       └── 01-push-dockerhub.yaml
            ├── hello-app/
            │       └── main/
            │           ├── main.py
            │           ├── Dockerfile
            │           └── requirements.txt

[ArgoCD - Kubernet/](https://github.com/josewolffsantiago/CompassUOL-CI-CD-ArgoCD-Sprint6-PBJUN2025-DevSecOPs)
            
            ├── README.md
            ├── gitops-microservices
            │    └── k8s
            │        ├── deployment.yaml
            │        └── service.yaml

### 2.1. Repositório Git (GitHub):

 Atua como a fonte da verdade (source of truth).

 Armazena os arquivos YAML do Kubernetes que descrevem todos os recursos da aplicação Online Boutique (Deployments, Services, etc.).

 Toda alteração no estado desejado da aplicação deve ser feita através de um commit neste repositório.

### 2.2. Automação com GitHub Actions:

Atua como o motor da automação e da Integração Contínua (CI) no nosso fluxo de trabalho.

Trigger: É acionado automaticamente por um git push no repositório Git e garante que cada nova alteração no código seja processada.

A principal função do GitHub Actions neste fluxo é pegar o código-fonte da aplicação, construir uma imagem Docker e empacotar todas as dependências necessárias.

### 2.3. ArgoCD (Ferramenta de GitOps):

Atua como o controlador que automatiza o deploy.

Fica em execução dentro do cluster Kubernetes e monitora continuamente o repositório Git.

Ele compara o estado definido nos manifestos do Git (estado desejado) com o estado dos recursos que estão realmente em execução no cluster (estado atual).

Se houver uma divergência, o ArgoCD sincroniza automaticamente ou manualmente as mudanças necessárias no cluster para garantir que o estado atual corresponda ao estado desejado.

### 2.4. Cluster Kubernetes (Rancher Desktop):

É o ambiente de execução onde a aplicação é implantada.

Orquestra os containers dos múltiplos microserviços que compõem a aplicação Online Boutique.

Recebe as instruções declarativas (aplicação dos manifestos) do ArgoCD.

------------------

## 3. Guia de Execução Passo a Passo

Esta seção descreve as etapas para implantar a aplicação Online Boutique utilizando o fluxo de GitOps.

>[!NOTE]
>Esta documentação não inclui tutorial completo de instalação de todos os componentes. No item [Tecnologias Utilizadas](https://github.com/josewolffsantiago/CompassUOL-CI-CD-ArgoCD-Sprint6-PBJUN2025-DevSecOPs?tab=readme-ov-file#11-tecnologias-utilizadas) tem todos os links para a instalação e configuração correta de cada componente para a Execução deste projeto.

### 3.1. Preparação do Repositório Git

Neste desafio, iremos fazer dois repositórios GIT. Pode verificar que você está seguindo a documentação de um repositório. Talvez você tenha chegado aqui através do outro repositório, que tem o link para este. Ou existe uma probabilidade grande de você nem saber da existência do outro repositório.

Pois bem, um repositório irá acontecer o CI utilizando o GitHub Actions, onde estarão os arquivos de Workflow do GitHub Actions, os dados da aplicação e os arquivos do Docker e o outro repositório é para o CD, mas precisamente para usarmos o ArgoCD para lançar a nossa aplicação. 

[Este repositório do ArgoCD](https://github.com/josewolffsantiago/CompassUOL-CI-CD-ArgoCD-Sprint6-PBJUN2025-DevSecOPs)

[Repositório do GitHub Actions](https://github.com/josewolffsantiago/CompassUOL-CI-CD-com-o-Github-Actions-Sprint6-PBJUN2025-DevSecOPs)

### 3.2. Instalação do ArgoCD no Cluster

No terminal, crie um namespace para a instalação do ArgoCD

            kubectl create namespace argocd

Abaixo, aplique o link da instalação 

            kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Verificação: Aguarde alguns minutos e verifique se os pods do ArgoCD estão em execução:


            kubectl get pods -n argocd

### 3.3. Acesso à Interface Web do ArgoCD

Para acessar a interface do ArgoCD, que por padrão não é exposta externamente, execute o comando:

            kubectl port-forward svc/argocd-server -n argocd 8080:443

Obter a Senha Inicial: A senha inicial do usuário admin é gerada automaticamente e armazenada em um Secret. Para obtê-la, execute:

            kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

Login: Acesse https://localhost:8080 em seu navegador. Use admin como usuário e a senha obtida no passo anterior.

### 3.4. Configurando o Dockerfile com a criação da nossa aplicação "Hello World" em Python

Precisamos dockerizar a nossa aplicação. Estes arquivos estão todos prontos e estão no outro repositório: [Repositório do GitHub Actions](https://github.com/josewolffsantiago/CompassUOL-CI-CD-com-o-Github-Actions-Sprint6-PBJUN2025-DevSecOPs). 

#### 3.4.1. Criação do main.py

Apenas um arquivo com um Hello Word em Python. Salve ele com o nome main.py na pasta ~/hello-app/main


            from fastapi import FastAPI
            app = FastAPI()
            @app.get("/")
            async def root():
                return {"message": "Hello World v3"}

Verifica que ele carrega um Framework chamado FastApi para a execução deste código. Graças ao Kubernets, começando primeiramente pelo Docker, não será necessário instalar nada localmente, além dos componentes acima

#### 3.4.2. Arquivo Dockerfile

Crie um arquivo na 
