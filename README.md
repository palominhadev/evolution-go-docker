# **Guia de Implantação Docker \- Evolution API (Go Edition)**

Este guia fornece o passo a passo completo para implantar a **Evolution API (Versão Go)** em conjunto com o **Postgres** e o **NATS Server** utilizando o Docker.

## **📋 Visão Geral da Arquitetura**

* **Evolution API:** O serviço principal da API do WhatsApp, desenvolvido em Go, focado em alta performance e baixo consumo de recursos.  
* **Postgres:** Banco de dados relacional utilizado para persistir as informações das instâncias, tokens e configurações do sistema.  
* **NATS Server:** Um message broker (mensageria) de alta performance utilizado para distribuição de eventos internos e escalabilidade horizontal.

## **🛠️ Pré-requisitos**

* Docker instalado e em execução no sistema operacional.  
* Um container PostgreSQL ativo e nomeado como postgres.  
* Um arquivo .env configurado no mesmo diretório onde os comandos serão executados.

## **🚀 Passo a Passo da Implantação**

### **1\. Criar uma Rede Docker Dedicada**

Criamos uma rede isolada do tipo *bridge* para garantir que todos os containers se comuniquem de forma segura e rápida através do DNS interno do Docker.  
```bash
docker network create rede-evolution
```

### **2\. Conectar o Container Postgres Existente**

Se você já possui um banco de dados Postgres rodando no Docker, conecte-o à nova rede criada para permitir que a API se comunique com ele sem a necessidade de reiniciar o container do banco.
```bash 
docker network connect rede-evolution postgres
```

### **3\. Executar o Servidor NATS**

O NATS Server gerencia os eventos da API. Executamos o container em segundo plano (-d) e configuramos a reinicialização automática caso o sistema falhe.  

```bash
docker container run -d --name nats-server --network rede-evolution \
--restart unless-stopped -p 4222:4222 nats:latest
```
### **4\. Executar a Evolution API**

Agora, iniciamos a Evolution API passando o arquivo de variáveis de ambiente (--env-file) e mapeando um volume persistente (-v) para garantir que as sessões e QR Codes do WhatsApp não sejam perdidos ao reiniciar o container.

```bash
docker container run -it -p 8080:8080 --name evolution_api \
--restart unless-stopped --network rede-evolution \
--env-file .env -v evolution_go_instances:/app/instances \
evoapicloud/evolution-go:latest
```

## **⚙️ Configuração do Arquivo .env**

Para que a aplicação funcione corretamente, certifique-se de que o seu arquivo .env contenha as strings de conexão apontando para os nomes dos containers dentro da rede Docker (postgres e nats-server funcionam como hostnames graças à rede compartilhada):  

# Configurações do Servidor
`SERVER_PORT=8080`  
`SERVER_URL=http://localhost:8080`

# Conexão com o Banco de Dados (Apontando para o container 'postgres')
`DATABASE_CONNECTION_URI=postgres://usuario:senha@container-postgres:5432/nome_do_banco?sslmode=disable`

# Configuração do NATS (Apontando para o container 'nats-server')
`NATS_URL=nats://nats-server:4222`
`NATS_URI=nats://nats-server:4222`

# AUTENTICAÇÃO E SEGURANÇA
`GLOBAL_API_KEY=suachaveseguraAQUI`

## **💡 Informações Adicionais e Boas Práticas**

* **Persistência de Dados:** O parâmetro \-v evolution\_go\_instances:/app/instances é crucial. Sem ele, toda vez que o container for atualizado ou recriado, todos os WhatsApps conectados serão deslogados.  
* **Logs da API:** Caso queira rodar a API em segundo plano (liberando o terminal), substitua o parâmetro \-it por \-d no comando do passo 4\. Para visualizar os logs posteriormente, utilize:

```bash
docker logs -f evolution_api
```

## **❓ Dúvidas e Documentação Oficial**

Para configurações avançadas de webhooks, documentação completa dos endpoints e atualizações, acesse o repositório oficial do projeto:  
👉 [Evolution Foundation GitHub \- evolution-go](https://github.com/evolution-foundation/evolution-go)