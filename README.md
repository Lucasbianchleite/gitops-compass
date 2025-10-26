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

## 🧪 tarefas extra 1
Vamos testar o ciclo completo do GitOps alterando o número de réplicas do frontend.


1.  **Altere o Manifesto**
    Abra o arquivo `k8s/online-boutique.yaml` e localize o `Deployment` com o nome `frontend`. Adicione e altere o campo `replicas` de `1` para `3`.


     ![Editando réplicas](https://github.com/user-attachments/assets/53bb61b3-d2b8-492c-847e-f7c027cff62d )



2.  **Observe a Mágica!** ✨
    Volte para a UI do ArgoCD. Em poucos instantes, ele detectará a alteração, entrará em estado `Syncing` e aplicará a mudança no cluster.



      ![ArgoCD Sincronizando](https://github.com/user-attachments/assets/333257a8-eb67-440c-a64a-6297bbed16c9)

6.  **Confirme no Cluster**
    Verifique os pods do frontend. Agora você verá 3 pods em execução!
    ```bash
    kubectl get pods -l app=frontend
    ```
    ![3 réplicas](https://github.com/user-attachments/assets/795df9f4-40f6-4d3a-8c00-394ae9c28bb4)

Este teste confirma que o Git é a única fonte da verdade e o ArgoCD garante que o estado do cluster corresponda ao que está versionado.

---

## 🧪 tarefas extra 2
# 🔐 Conectando o ArgoCD a Repositórios Git Privados

Em um ambiente de produção, manter o código-fonte e os manifestos de configuração em **repositórios privados** é uma prática de segurança fundamental.  
Para que uma ferramenta de **GitOps** como o **ArgoCD** possa automatizar o deploy, ela precisa de um meio seguro para acessar esses repositórios.

Este guia detalha **dois métodos robustos e amplamente utilizados** para estabelecer essa conexão:

1. Usando um **Personal Access Token (PAT)** sobre HTTPS.  
2. Configurando o acesso via **chaves SSH**.

---

## 🚀 Método 1: Conexão via Personal Access Token (HTTPS)

Esta abordagem é ideal para uma configuração rápida e quando a política de rede favorece o tráfego HTTPS.  
Um **PAT** funciona como uma senha com escopo limitado, permitindo um controle granular sobre as permissões.

---

### 🧩 Passo a Passo

#### 1️⃣ Gerar o Personal Access Token (PAT)

O primeiro passo é criar um token no seu provedor Git (GitHub, GitLab, Bitbucket, etc.).

**No GitHub:**

1. Vá para:  
   `Settings > Developer settings > Personal access tokens > Tokens (classic)`
2. Clique em **Generate new token**.
3. Dê um nome descritivo, como `argocd-read-only-token`.
4. **Expiration:** Defina uma data de expiração — **nunca use tokens permanentes**.
5. **Scopes:** Selecione apenas o escopo `repo`.
6. Clique em **Generate token** e **copie o token imediatamente** — ele **não será exibido novamente**.

---

#### 2️⃣ Montar a URL do Repositório

Com o token em mãos, construa a URL de clone no seguinte formato:

```bash
https://<SEU_TOKEN>@github.com/<NOME_DE_USUARIO>/<NOME_DO_REPOSITORIO>.git
```

> Substitua `<SEU_TOKEN>`, `<NOME_DE_USUARIO>` e `<NOME_DO_REPOSITORIO>` pelos valores reais.

---

#### 3️⃣ Adicionar o Repositório no ArgoCD

Ao criar uma nova aplicação no ArgoCD (pela UI ou via manifesto YAML), use a URL acima em:

```yaml
spec:
  source:
    repoURL: "https://<SEU_TOKEN>@github.com/<NOME_DE_USUARIO>/<NOME_DO_REPOSITORIO>.git"
```

O ArgoCD usará o token embutido na URL para se autenticar e acessar o repositório.

> ⚠️ **Alerta de Segurança:**  
> Trate seu PAT como uma senha. Nunca o exponha em logs ou arquivos públicos.  
> Use escopos limitados e datas de expiração para reduzir riscos.

---

## 🔑 Método 2: Conexão via Chaves SSH

O acesso via SSH é o método **preferido para automação e comunicação segura entre sistemas**.  
Ele utiliza um par de chaves criptográficas (pública e privada) para autenticação, eliminando a necessidade de senhas ou tokens.

---

### 🧩 Passo a Passo

#### 1️⃣ Gerar um Par de Chaves SSH

É uma boa prática gerar um par de chaves dedicado exclusivamente para o ArgoCD.

Execute o comando abaixo no terminal para gerar uma chave moderna e segura (ed25519):

```bash
ssh-keygen -t ed25519 -C "argocd-key" -f ./argocd_id_ed25519
```

> Quando o prompt solicitar uma **passphrase**, pressione **Enter** duas vezes para deixá-la em branco.  
> O ArgoCD precisa da chave **sem senha** para operar automaticamente.

Isso criará dois arquivos:

- `argocd_id_ed25519` → 🔒 chave **privada**
- `argocd_id_ed25519.pub` → 🔑 chave **pública**

---

#### 2️⃣ Adicionar a Chave Pública ao Repositório (Deploy Key)

A chave pública autoriza o ArgoCD a acessar o repositório.

1. Copie o conteúdo da chave pública:
   ```bash
   cat ./argocd_id_ed25519.pub
   ```
2. No GitHub, vá para:  
   `Settings > Deploy Keys > Add deploy key`
3. **Title:** `ArgoCD`
4. **Key:** cole o conteúdo copiado.
5. **Allow write access:** deixe **desmarcado** (somente leitura).
6. Clique em **Add key**.

---

#### 3️⃣ Adicionar a Chave Privada ao ArgoCD

Agora, forneça a chave privada ao ArgoCD para autenticação.

1. Acesse a **interface web** do ArgoCD.  
2. Vá para **Settings > Repositories**.  
3. Clique em **+ CONNECT REPO**.  
4. Preencha os campos:

| Campo | Valor |
|-------|-------|
| **CONNECTION METHOD** | SSH |
| **NAME** | `meu-repo-privado-ssh` |
| **PROJECT** | O projeto do ArgoCD que usará o repositório |
| **REPOSITORY URL** | `git@github.com:<NOME_DE_USUARIO>/<NOME_DO_REPOSITORIO>.git` |
| **SSH PRIVATE KEY** | Conteúdo da sua `argocd_id_ed25519` |

Obtenha o conteúdo da chave privada com:

```bash
cat ./argocd_id_ed25519
```

Após preencher, clique em **CONNECT**.

Se tudo estiver correto, o status aparecerá como ✅ **Successful**.

---

## 🧠 Conclusão

Com isso, o **ArgoCD** está pronto para **monitorar e sincronizar aplicações** a partir de um **repositório privado** de forma **segura, automatizada e auditável**.

> 💡 **Dica:** Prefira o método SSH em ambientes produtivos, pois ele elimina a exposição de tokens sensíveis em configurações YAML.




---




