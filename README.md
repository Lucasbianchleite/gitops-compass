# 📥 GitOps na Prática — Online Boutique com ArgoCD e Kubernetes

### 🛠 Tecnologias Utilizadas

![ARGOCD](https://img.shields.io/badge/ArgoCD-EB6E34?style=for-the-badge&logo=argo&logoColor=white)
![KUBERNETES](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![RANCHER](https://img.shields.io/badge/Rancher-0075A8?style=for-the-badge&logo=rancher&logoColor=white)
![GITOPS](https://img.shields.io/badge/GitOps-Automation-1F6FEB?style=for-the-badge&logo=git&logoColor=white)
![DOCKER](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## 📌 Descrição do Projeto

Este projeto é uma **imersão prática em GitOps e Kubernetes**, usando **Rancher Desktop** como cluster local e **ArgoCD** para automação de deploys.  

Ele executa os **microserviços da Online Boutique** de forma **automatizada e controlada via Git**, permitindo que todas as alterações no repositório reflitam automaticamente no cluster.  

O objetivo é aprender como gerenciar aplicações distribuídas de forma **segura, rastreável e escalável**, seguindo práticas modernas de **entrega contínua em ambientes cloud-native**.

## Requisitos para o ambiente

| Requisito                 | Descrição                                                                 | Observação / Alternativa                  |
|----------------------------|---------------------------------------------------------------------------|------------------------------------------|
| Git                        | Necessário para versionamento e controle do código.                      | —                                        |
| Conta no GitHub            | Para hospedar repositórios e integrar com o ArgoCD.                       | —                                        |
| Docker ou containerd       | Para criar e gerenciar containers localmente.                             | —                                        |
| ArgoCD CLI                 | Usada para configurar aplicações via acesso local                         | Também é possível usar a CLI via argoCD cli.      |
| Kubectl                    | Ferramenta de linha de comando para interagir com o cluster Kubernetes.   | —                                        |
| Rancher Desktop            | Deve estar instalado com Kubernetes ativo.                                | Pode-se usar Minikube ou Kind para testes locais. |






