# Agência de Viagem AI

API de um assistente de agência de viagens construída com **Quarkus** e **LangChain4j**. O assistente responde perguntas em linguagem natural usando um modelo de linguagem local (via **Ollama**) e consulta o catálogo de pacotes da agência por meio de **RAG** (Easy RAG).

## Tecnologias

- Java + [Quarkus](https://quarkus.io/) 3.40
- [Quarkus LangChain4j](https://docs.quarkiverse.io/quarkus-langchain4j/dev/index.html) (Ollama + Easy RAG)
- [Ollama](https://ollama.com/) rodando localmente
- Maven (wrapper incluso)

## Pré-requisitos

- JDK instalado (a versão exigida está no `pom.xml`)
- [Ollama](https://ollama.com/) instalado
- Modelo baixado:

```shell
ollama pull gemma3:4b
```

> **Sobre o modelo:** o projeto usa o `gemma3:4b` (~3 GB). Modelos maiores, como o `gpt-oss:20b`, ficaram lentos demais em máquinas sem GPU e com ~16 GB de RAM, e estouraram o timeout das requisições.

## Como executar

1. Inicie o servidor do Ollama (em um terminal separado):

```shell
ollama serve
```

2. Inicie a aplicação em modo dev:

```shell
./mvnw compile quarkus:dev
```

A aplicação sobe em `http://localhost:8080`. A Dev UI fica em <http://localhost:8080/q/dev/>.

## Uso da API

### `POST /travel`

Recebe uma pergunta em texto puro e devolve a resposta do assistente em texto puro.

```shell
curl -X POST -H "Content-Type: text/plain" \
  -d "Estou planejando uma viagem para o Japão. Qual a melhor época do ano para visitar e por quê? Responda em português" \
  http://localhost:8080/travel
```

> **PowerShell:** `curl` é um alias de `Invoke-WebRequest` e não aceita essas opções. Use `curl.exe` ou rode o comando em um terminal bash (Git Bash).

> **Tempo de resposta:** sem GPU, o modelo roda na CPU e uma resposta pode levar cerca de 1 minuto. A primeira chamada após iniciar o Ollama (ou depois de um período ocioso) demora mais, porque o modelo precisa ser carregado na memória.

## Configuração

Arquivo: `src/main/resources/application.properties`

| Propriedade | Valor | Descrição |
|---|---|---|
| `quarkus.langchain4j.ollama.base-url` | `http://localhost:11434/` | URL do servidor Ollama |
| `quarkus.langchain4j.ollama.chat-model.model-id` | `gemma3:4b` | Modelo usado nas conversas |
| `quarkus.langchain4j.ollama.timeout` | `240s` | Tempo máximo de espera por resposta do modelo |
| `quarkus.langchain4j.easy-rag.path` | `src/main/resources/rag` | Pasta com os documentos lidos pelo RAG |

## Estrutura do projeto

```
src/main/java/dev/ia/
├── TravelAgentAssistant.java   # AI Service (LangChain4j) que conversa com o modelo
└── TravelAgentResource.java    # Endpoint REST POST /travel
src/main/resources/
├── application.properties      # Configuração da aplicação
└── rag/
    └── pacotes-viagem.md       # Catálogo de pacotes usado como base de conhecimento (RAG)
```

## Base de conhecimento (RAG)

Na inicialização, o Easy RAG lê os arquivos da pasta `src/main/resources/rag`, gera os embeddings e os usa para enriquecer as respostas do assistente. Para adicionar ou alterar pacotes, edite ou crie arquivos nessa pasta e reinicie a aplicação.

Pacotes disponíveis atualmente: **Aventura Amazônia** e **Tesouros do Egito**.

## Empacotamento

```shell
./mvnw package
java -jar target/quarkus-app/quarkus-run.jar
```

Para gerar um executável nativo (requer GraalVM ou build em contêiner), veja o [guia do Quarkus](https://quarkus.io/guides/maven-tooling).
