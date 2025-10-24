# 📥 GitOps na Prática: Online Boutique com ArgoCD e Kubernetes

Este projeto é um guia prático para implementar a aplicação **Online Boutique** (um sistema de microserviços do Google) em um ambiente Kubernetes local, utilizando **ArgoCD** para orquestrar o deploy de forma totalmente automatizada através de um fluxo GitOps.

### 🛠️ Tecnologias Utilizadas

![ArgoCD](https://img.shields.io/badge/ArgoCD-EB6E34?style=for-the-badge&logo=argo&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Rancher](https://img.shields.io/badge/Rancher-0075A8?style=for-the-badge&logo=rancher&logoColor=white)
![GitOps](https://img.shields.io/badge/GitOps-Automation-1F6FEB?style=for-the-badge&logo=git&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## 📖 Sumário

* [Visão Geral do Projeto](#-visão-geral-do-projeto)
* [Pré-requisitos](#-pré-requisitos)
* [🚀 Guia de Implementação](#-guia-de-implementação)
    * [Etapa 1: Preparar o Repositório de Manifestos](#etapa-1--preparar-o-repositório-de-manifestos)
    * [Etapa 2: Instalar e Configurar o ArgoCD](#etapa-2--instalar-e-configurar-o-argocd)
    * [Etapa 3: Criar a Aplicação no ArgoCD](#etapa-3--criar-a-aplicação-no-argocd)
    * [Etapa 4: Acessar a Aplicação](#etapa-4--acessar-a-aplicação)
* [🧪 Validando o Fluxo GitOps](#-validando-o-fluxo-gitops)
* [🧹 Limpeza do Ambiente](#-limpeza-do-ambiente)

---

## 🎯 Visão Geral do Projeto

O objetivo deste laboratório é demonstrar como gerenciar uma aplicação baseada em microserviços de forma **segura, rastreável e escalável**, seguindo as melhores práticas de entrega contínua em ambientes cloud-native.

Ao final, você terá aprendido a:
- **Configurar um repositório Git** como única fonte da verdade para o estado da sua aplicação.
- **Instalar e configurar o ArgoCD** para monitorar o repositório.
- **Automatizar o deploy** de uma aplicação complexa no Kubernetes.
- **Validar o ciclo GitOps**, onde uma alteração no Git reflete automaticamente no ambiente de execução.

## ✅ Pré-requisitos

Garanta que você tenha as seguintes ferramentas instaladas e configuradas em sua máquina.

| Ferramenta | Descrição | Instalação |
| :--- | :--- | :--- |
| **Git** | Essencial para o versionamento do código. | [git-scm.com](https://git-scm.com/downloads) |
| **GitHub CLI** (Opcional) | Facilita a criação de repositórios e forks. | [cli.github.com](https://cli.github.com/) |
| **Docker / containerd** | Engine de contêineres para o cluster local. | [docker.com](https://www.docker.com/products/docker-desktop/) |
| **Rancher Desktop** | Ambiente Kubernetes local. | [rancherdesktop.io](https://rancherdesktop.io/) |
| **kubectl** | CLI para interagir com o cluster Kubernetes. | [kubernetes.io](https://kubernetes.io/docs/tasks/tools/) |
| **ArgoCD CLI** | CLI para gerenciar o ArgoCD. | [argo-cd.readthedocs.io](https://argo-cd.readthedocs.io/en/stable/cli_installation/) |

---

## 🚀 Guia de Implementação

### Etapa 1: 📂 Preparar o Repositório de Manifestos

O Git será nossa fonte da verdade. Vamos preparar um repositório para armazenar os manifestos Kubernetes.

1.  **Faça um Fork do Projeto Original**
    Crie uma cópia (fork) do repositório da aplicação para a sua conta no GitHub:
    * **Repositório Original:** [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)

2.  **Crie um Novo Repositório para os Manifestos**
    Crie um **novo repositório** em sua conta (ex: `online-boutique-manifests`). Este repositório conterá apenas os arquivos YAML que o ArgoCD irá monitorar.

3.  **Adicione o Manifesto do Kubernetes**
    No seu **novo** repositório, crie a seguinte estrutura de arquivos, copiando o conteúdo de `release/kubernetes-manifests.yaml` do projeto original.

    ```sh
    .
    └── k8s/
        └── online-boutique.yaml
    ```

### Etapa 2: 📦 Instalar e Configurar o ArgoCD

Agora, vamos instalar o ArgoCD no nosso cluster Kubernetes local.

1.  **Verifique a Conexão com o Cluster**
    Garanta que seu `kubectl` está configurado para o cluster do Rancher Desktop.
    ```bash
    kubectl get nodes
    ```
    <details>
      <summary>Clique para ver a saída esperada</summary>
    
      ```
      NAME              STATUS   ROLES                    AGE   VERSION
      rancher-desktop   Ready    control-plane,master   1d    v1.29.1+k3s1
      ```
    </details>

2.  **Instale o ArgoCD**
    Crie o namespace `argocd` e aplique os manifestos de instalação oficiais.
    ```bash
    # Cria o namespace dedicado
    kubectl create namespace argocd

    # Aplica os manifestos de instalação do ArgoCD
    kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
    ```

3.  **Acesse a Interface Web (UI)**
    Para acessar o painel web, exponha o serviço `argocd-server` localmente em uma nova janela de terminal.
    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
    Acesse a UI em **[https://localhost:8080](https://localhost:8080)** (ignore o aviso de certificado).

4.  **Obtenha a Senha de Administrador** 🔑
    O usuário padrão é `admin`. Obtenha a senha inicial com o comando:
    ```bash
    # Decodifica a senha do secret do Kubernetes
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

### Etapa 3: ✨ Criar a Aplicação no ArgoCD

Vamos conectar o ArgoCD ao nosso repositório de manifestos.

1.  **Login via CLI**
    Use a senha obtida no passo anterior para fazer login.
    ```bash
    argocd login localhost:8080 --insecure
    ```

2.  **Criação da Aplicação**
    Execute o comando abaixo, **substituindo `<SEU_USUARIO_GITHUB>/<SEU_REPO_MANIFESTOS>`** pelos dados do seu repositório.
    ```bash
    argocd app create online-boutique \
      --repo [https://github.com/](https://github.com/)<SEU_USUARIO_GITHUB>/<SEU_REPO_MANIFESTOS>.git \
      --path k8s \
      --dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) \
      --dest-namespace default \
      --revision HEAD \
      --sync-policy automated
    ```
    **O que cada flag faz?**
    * `--repo`: URL do seu repositório de manifestos.
    * `--path`: Pasta dentro do repositório onde os manifestos se encontram.
    * `--dest-server`: Cluster de destino (o cluster interno do K8s).
    * `--dest-namespace`: Namespace onde a aplicação será implantada.
    * `--sync-policy automated`: Habilita a sincronização automática. Qualquer `git push` no repositório irá disparar um deploy.

### Etapa 4: 🌐 Acessar a Aplicação

> **Nota Importante:** O serviço `frontend-external` é do tipo `LoadBalancer`. Em ambientes locais, ele ficará com o status `<pending>`, pois não há um provedor de nuvem para atribuir um IP externo. Isso é esperado.

1.  **Acesso via Port-Forward**
    Para acessar a loja, crie um túnel local para o serviço do frontend:
    ```bash
    kubectl port-forward svc/frontend-external 8000:80 -n default
    ```

2.  **Visite a Loja!**
    Abra seu navegador e acesse: **🔗 [http://localhost:8000](http://localhost:8000)**

Parabéns! A aplicação Online Boutique está rodando no seu cluster, gerenciada pelo ArgoCD.

![Online Boutique UI](https://github.com/user-attachments/assets/aa9bee1e-c9ce-4d8b-933e-30fa13baae6f)

---

## 🧪 Validando o Fluxo GitOps

Vamos testar o ciclo completo do GitOps alterando o número de réplicas do frontend.

1.  **Clone o Repositório de Manifestos**
    Se ainda não o fez, clone o repositório que você criou na Etapa 1.
    ```bash
    git clone [https://github.com/](https://github.com/)<SEU_USUARIO_GITHUB>/<SEU_REPO_MANIFESTOS>.git
    cd <SEU_REPO_MANIFESTOS>
    ```

2.  **Altere o Manifesto**
    Abra o arquivo `k8s/online-boutique.yaml` e localize o `Deployment` com o nome `frontend`. Altere o campo `replicas` de `1` para `3`.
    ![Editando réplicas](https://github.com/user-attachments/assets/9ea2aa06-f6b7-4c0d-b33c-38c9757adad4)

3.  **Faça o Commit e Push**
    Envie a alteração para o GitHub.
    ```bash
    git add .
    git commit -m "feat: Aumenta réplicas do frontend para 3"
    git push
    ```

4.  **Observe a Mágica!** ✨
    Volte para a UI do ArgoCD. Em poucos instantes, ele detectará a alteração, entrará em estado `Syncing` e aplicará a mudança no cluster.
    ![ArgoCD Sincronizando](https://github.com/user-attachments/assets/45116b6c-7eb9-463a-8d83-f47b9c31f543)

5.  **Confirme no Cluster**
    Verifique os pods do frontend. Agora você verá 3 pods em execução!
    ```bash
    kubectl get pods -l app=frontend
    ```
    ![3 réplicas](https://github.com/user-attachments/assets/b5390dc0-599d-483a-88a8-8aa336d101e0)

Este teste confirma que o Git é a única fonte da verdade e o ArgoCD garante que o estado do cluster corresponda ao que está versionado.

---

## 🧹 Limpeza do Ambiente

Para remover os recursos criados, siga os passos abaixo:

1.  **Exclua a Aplicação no ArgoCD**
    ```bash
    argocd app delete online-boutique
    ```

2.  **Desinstale o ArgoCD do Cluster**
    ```bash
    kubectl delete -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
    kubectl delete namespace argocd
    ```

3.  **Pare os `port-forwards`**
    Volte para os terminais onde os comandos `kubectl port-forward` estão rodando e pressione `Ctrl + C` para encerrá-los.
