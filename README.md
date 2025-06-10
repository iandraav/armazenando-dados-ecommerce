# 🚀 Desafio DIO: Armazenando Dados de um E-Commerce na Cloud 🛒

Este projeto foi desenvolvido como parte do Bootcamp "Microsoft Azure AI Fundamentals" da Digital Innovation One (DIO), com o objetivo de demonstrar a capacidade de armazenar e gerenciar dados de um e-commerce utilizando serviços de plataforma do Microsoft Azure.

---

## 🎯 Objetivo do Projeto

O desafio consistiu em simular uma arquitetura para armazenamento de dados de um e-commerce na nuvem, aplicando os conceitos de contêineres e bancos de dados gerenciados, focando em simplicidade e escalabilidade.

---

## 💻 Tecnologias Utilizadas

* **Azure Services:**
    * [Azure Container Apps](https://azure.microsoft.com/services/container-apps/) - Para hospedar a API de e-commerce conteinerizada.
    * [Azure Container Registry (ACR)](https://azure.microsoft.com/services/container-registry/) - Para armazenar e gerenciar as imagens Docker da aplicação.
    * [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) - Para o armazenamento persistente dos dados relacionais do e-commerce (ex: produtos, pedidos).
    * Azure Resource Groups - Para organizar os recursos.
* **Ferramentas & Linguagens:**
    * [Docker](https://www.docker.com/) - Para conteinerização da aplicação.
    * [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) - Para interagir com os serviços Azure via linha de comando.
    * [Python 3.x](https://www.python.org/) - Linguagem de programação utilizada para a API.
    * [Flask](https://flask.palletsprojects.com/en/2.0.x/) - Framework web Python para a API.
    * [Git & GitHub](https://github.com/) - Para controle de versão e hospedagem do repositório.

---

## 🏛️ Arquitetura da Solução

A solução proposta consiste em uma API RESTful, desenvolvida em Python e conteinerizada, que interage com um banco de dados Azure SQL Database para armazenar informações de produtos de um e-commerce. A imagem do contêiner é gerenciada no Azure Container Registry e implantada no Azure Container Apps, garantindo escalabilidade e facilidade de gerenciamento.

![Diagrama da Arquitetura](link_para_sua_imagem_de_diagrama.png)

_Breve descrição do fluxo:_ Um cliente (simulado por uma requisição HTTP) envia dados para a API hospedada no Azure Container Apps. A API processa esses dados e os persiste no Azure SQL Database. A imagem do contêiner da API é armazenada no Azure Container Registry.

---

## 🚀 Passos para a Implementação

Aqui estão os principais passos seguidos para implementar esta solução:

1.  **Criação do Grupo de Recursos:**
    ```bash
    az group create --name rg-ecommerce-dio --location eastus
    ```
2.  **Configuração do Azure SQL Database:**
    * Criação do servidor SQL e do banco de dados `db-ecommerce`.
    * Configuração da regra de firewall para permitir acesso dos serviços Azure.
    ```bash
    # Exemplo: Criar servidor SQL
    az sql server create --name sqlserver-ecommerce-dio --resource-group rg-ecommerce-dio --location eastus --admin-user <seu_admin> --admin-password <sua_senha_segura>
    # Exemplo: Criar banco de dados
    az sql db create --resource-group rg-ecommerce-dio --server sqlserver-ecommerce-dio --name db-ecommerce --edition GeneralPurpose --family Gen5 --capacity 2
    # Exemplo: Configurar firewall para permitir acesso Azure
    az sql server firewall-rule create --resource-group rg-ecommerce-dio --server sqlserver-ecommerce-dio --name AllowAzureServices --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```
    * (Detalhe: Criada uma tabela `Products` com colunas `Id`, `Name`, `Price`.)

3.  **Criação do Azure Container Registry (ACR):**
    ```bash
    az acr create --resource-group rg-ecommerce-dio --name acrecommercedio --sku Basic
    ```
4.  **Desenvolvimento e Conteinerização da API (Python Flask):**
    * Um `Dockerfile` foi criado para construir a imagem da API.
    * O código da API (`app.py`) processa requisições POST para `/products` e insere dados no SQL Database.
    * _Exemplo de `Dockerfile` e `app.py` podem ser encontrados na raiz deste repositório._

5.  **Construção e Push da Imagem Docker para o ACR:**
    ```bash
    # Logar no ACR
    az acr login --name acrecommercedio
    # Construir e taggear a imagem
    docker build -t acrecommercedio.azurecr.io/ecommerce-api:v1 .
    # Fazer push da imagem
    docker push acrecommercedio.azurecr.io/ecommerce-api:v1
    ```
6.  **Implantação no Azure Container Apps:**
    * Criação do ambiente de Container Apps.
    * Implantação do Container App puxando a imagem do ACR.
    * A string de conexão do banco de dados foi configurada como um **segredo** para maior segurança.
    ```bash
    # Exemplo: Criar ambiente de Container Apps
    az containerapp env create --name appenv-ecommerce --resource-group rg-ecommerce-dio --location eastus
    # Exemplo: Criar Container App com segredo
    az containerapp create --name api-ecommerce-dio --resource-group rg-ecommerce-dio --environment appenv-ecommerce \
      --image acrecommercedio.azurecr.io/ecommerce-api:v1 \
      --target-port 80 --ingress external --query properties.latestRevisionFqdn \
      --secrets "db-connection-string=<Sua_Connection_String_do_SQL_Database>" \
      --env-vars "DB_CONNECTION_STRING=secretref:db-connection-string"
    ```
7.  **Teste da API:**
    * Utilizado o Postman para enviar uma requisição POST para a URL do Container App (ex: `https://api-ecommerce-dio.<unique-id>.azurecontainerapps.io/products`).

---

## 📸 Screenshots

Aqui estão algumas capturas de tela dos recursos implantados no Azure:

* **Visão Geral do Azure Container App:**
    ![Visão Geral Container App](assets/containerapp_overview.png)
* **Configuração de Segredos no Container App:**
    ![Configuração de Segredos](assets/containerapp_secrets.png)
* **Visão Geral do Azure SQL Database:**
    ![Visão Geral SQL Database](assets/sqldb_overview.png)
* **Azure Container Registry - Repositórios:**
    ![ACR Repositórios](assets/acr_repositories.png)
* **Teste da API com Postman:**
    ![Teste API Postman](assets/api_postman_test.png)

---

## 🧠 Insights e Aprendizados

Durante a execução deste desafio, pude aprofundar meu conhecimento e praticar os seguintes conceitos:

* **Poder dos Serviços PaaS:** O Azure Container Apps demonstrou como é possível implantar aplicações conteinerizadas de forma rápida, sem gerenciar a infraestrutura subjacente de Kubernetes, abstraindo a complexidade e permitindo foco no desenvolvimento.
* **Gerenciamento de Imagens com ACR:** O Azure Container Registry é fundamental para armazenar, gerenciar e versionar imagens Docker de forma segura e integrada ao ecossistema Azure.
* **Segurança de Segredos:** A importância de utilizar o gerenciamento de segredos nos Container Apps (e não variáveis de ambiente comuns) para proteger informações sensíveis como strings de conexão de banco de dados.
* **Conectividade entre Serviços:** Entendimento de como configurar permissões de rede e firewall para que um serviço conteinerizado possa se comunicar com um banco de dados gerenciado no Azure.
* **Escalabilidade e Revisões:** Compreendi a capacidade de escalabilidade automática do Container Apps e o conceito de revisões, que facilita atualizações seguras e rollbacks.
* **Desenvolvimento de APIs Leves:** Prática na criação de uma API REST simples que serve como interface para a camada de persistência de dados.

---

## 💡 Possibilidades de Melhoria e Próximos Passos

Este projeto básico pode ser expandido de várias maneiras para simular um e-commerce mais robusto:

* **CI/CD:** Implementar um pipeline de CI/CD (ex: Azure DevOps Pipelines ou GitHub Actions) para automatizar o build da imagem Docker e a implantação no Azure Container Apps.
* **Autenticação e Autorização:** Adicionar mecanismos de segurança para a API.
* **Mensageria:** Integrar com Azure Service Bus ou Azure Event Hubs para processamento assíncrono de pedidos ou eventos (ex: processar pedidos após a compra).
* **Outros Bancos de Dados:** Explorar o uso de Azure Cosmos DB para o catálogo de produtos (NoSQL) ou Azure Cache for Redis para cache de sessão.
* **Monitoramento Avançado:** Configurar o Application Insights para telemetria detalhada da API.
* **Front-end:** Conectar uma aplicação front-end (ex: React, Angular)
