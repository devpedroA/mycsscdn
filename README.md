# Projeto Fastify com TypeScript, Zod e Swagger

Este projeto é um exemplo de como configurar um servidor Fastify com TypeScript, Zod para validação de dados e Swagger para documentação da API.

## Pré-requisitos

-   [Node.js](https://nodejs.org/) (versão 18 ou superior)
-   [npm](https://www.npmjs.com/) (geralmente instalado com o Node.js)

## Configuração

1.  **Inicialize o projeto**

    ```bash
    npm init -y
    ```

2.  **Instale as dependências**

    Execute o seguinte comando para instalar todas as dependências do projeto:

    ```bash
    npm install fastify zod @fastify/swagger @fastify/swagger-ui
    npm install -D typescript @types/node tsx
    ```

## Configuração do Projeto

### Dependências

As seguintes dependências são utilizadas neste projeto:

-   **fastify**: Framework web Node.js.
-   **zod**: Biblioteca para declaração e validação de esquemas.
-   **@fastify/swagger**: Plugin Fastify para gerar documentação Swagger.
-   **@fastify/swagger-ui**: Plugin Fastify para servir a interface do Swagger UI.
-   **typescript**: Linguagem de programação TypeScript.
-   **@types/node**: Definições de tipo para Node.js.
-   **tsx**: Executor de TypeScript para Node.js.

### Scripts

Os seguintes scripts estão definidos no `package.json`:

```json
{
  "name": "fastify-project",
  "version": "1.0.0",
  "description": "",
  "main": "dist/server.js",
  "scripts": {
    "dev": "tsx src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@fastify/swagger": "^8.14.0",
    "@fastify/swagger-ui": "^2.1.0",
    "fastify": "^4.26.2",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "@types/node": "^20.11.30",
    "tsx": "^4.7.1",
    "typescript": "^5.4.3"
  }
}
