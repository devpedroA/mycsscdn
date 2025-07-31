# Guia Completo: Fastify + TypeScript + Prisma + SOLID

Este guia apresenta uma estrutura completa para desenvolvimento de APIs usando Fastify com TypeScript, seguindo princípios SOLID e boas práticas de arquitetura.

## 📚 Índice

1. [Configuração Inicial](#configuração-inicial)
2. [Estrutura do Projeto](#estrutura-do-projeto)
3. [Configuração do Banco de Dados](#configuração-do-banco-de-dados)
4. [Implementação SOLID](#implementação-solid)
5. [Documentação da API](#documentação-da-api)
6. [Testes](#testes)
7. [Scripts Úteis](#scripts-úteis)

## 🚀 Configuração Inicial

### 1. Inicializar o projeto

```bash
npm init -y
```

### 2. Instalar dependências

**Dependências principais:**
```bash
npm install fastify @fastify/swagger @fastify/swagger-ui @prisma/client dotenv zod bcryptjs jsonwebtoken
```

**Dependências de desenvolvimento:**
```bash
npm install -D typescript @types/node @types/bcryptjs @types/jsonwebtoken tsx prisma vitest @vitest/ui
```

### 3. Configurar TypeScript

**tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "outDir": "dist",
    "rootDir": "src",
    "sourceMap": true,
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@/controllers/*": ["controllers/*"],
      "@/services/*": ["services/*"],
      "@/repositories/*": ["repositories/*"],
      "@/utils/*": ["utils/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 4. Package.json atualizado

```json
{
  "name": "fastify-api-solid",
  "version": "1.0.0",
  "description": "API REST com Fastify, TypeScript e arquitetura SOLID",
  "main": "dist/server.js",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:run": "vitest run",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "db:seed": "tsx src/prisma/seed.ts"
  },
  "keywords": ["fastify", "typescript", "prisma", "solid"],
  "author": "",
  "license": "ISC"
}
```

## 📁 Estrutura do Projeto

```
src/
├── controllers/           # Controladores das rotas
│   ├── user.controller.ts
│   └── auth.controller.ts
├── services/             # Regras de negócio (Use Cases)
│   ├── user/
│   │   ├── create-user.service.ts
│   │   ├── get-user.service.ts
│   │   └── update-user.service.ts
│   └── auth/
│       ├── authenticate.service.ts
│       └── register.service.ts
├── repositories/         # Camada de dados
│   ├── interfaces/
│   │   └── user.repository.interface.ts
│   ├── memory/          # Para testes
│   │   └── user.memory.repository.ts
│   └── prisma/          # Implementação real
│       └── user.prisma.repository.ts
├── factories/           # Factory pattern para injeção de dependência
│   ├── make-user.factory.ts
│   └── make-auth.factory.ts
├── dtos/               # Data Transfer Objects
│   ├── user.dto.ts
│   └── auth.dto.ts
├── utils/              # Utilitários
│   ├── password.utils.ts
│   └── jwt.utils.ts
├── routes/             # Definição das rotas
│   ├── user.routes.ts
│   └── auth.routes.ts
├── config/             # Configurações
│   ├── database.ts
│   └── swagger.ts
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── tests/              # Testes
│   ├── unit/
│   └── integration/
└── server.ts           # Arquivo principal
```

## 🗄️ Configuração do Banco de Dados

### 1. Docker Container (MariaDB)

```bash
docker run --name biblioteca-api-mysql \
  -e MARIADB_ROOT_PASSWORD=docker \
  -e MARIADB_DATABASE=biblioteca-api \
  -e MARIADB_USER=docker \
  -e MARIADB_PASSWORD=docker \
  -p 3306:3306 \
  -d bitnami/mariadb:latest
```

### 2. Variáveis de Ambiente (.env)

```env
# Database
DATABASE_URL="mysql://docker:docker@localhost:3306/biblioteca-api"

# JWT
JWT_SECRET="your-super-secret-jwt-key"
JWT_EXPIRES_IN="7d"

# Server
PORT=3000
NODE_ENV="development"
```

### 3. Schema Prisma

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("users")
}
```

## 🏗️ Implementação SOLID

### 1. DTOs (Data Transfer Objects)

```typescript
// src/dtos/user.dto.ts
import { z } from 'zod'

export const CreateUserDTO = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(6)
})

export const UpdateUserDTO = z.object({
  name: z.string().min(2).optional(),
  email: z.string().email().optional()
})

export const UserResponseDTO = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string(),
  createdAt: z.date()
})

export type CreateUserData = z.infer<typeof CreateUserDTO>
export type UpdateUserData = z.infer<typeof UpdateUserDTO>
export type UserResponse = z.infer<typeof UserResponseDTO>
```

### 2. Repository Interface

```typescript
// src/repositories/interfaces/user.repository.interface.ts
import { CreateUserData, UpdateUserData, UserResponse } from '@/dtos/user.dto'

export interface UserRepositoryInterface {
  create(data: CreateUserData): Promise<UserResponse>
  findById(id: string): Promise<UserResponse | null>
  findByEmail(email: string): Promise<UserResponse | null>
  update(id: string, data: UpdateUserData): Promise<UserResponse>
  delete(id: string): Promise<void>
  findMany(page: number, limit: number): Promise<UserResponse[]>
}
```

### 3. Repository Implementation (Prisma)

```typescript
// src/repositories/prisma/user.prisma.repository.ts
import { prisma } from '@/config/database'
import { UserRepositoryInterface } from '../interfaces/user.repository.interface'
import { CreateUserData, UpdateUserData, UserResponse } from '@/dtos/user.dto'

export class UserPrismaRepository implements UserRepositoryInterface {
  async create(data: CreateUserData): Promise<UserResponse> {
    const user = await prisma.user.create({
      data,
      select: {
        id: true,
        name: true,
        email: true,
        createdAt: true
      }
    })
    return user
  }

  async findById(id: string): Promise<UserResponse | null> {
    const user = await prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        name: true,
        email: true,
        createdAt: true
      }
    })
    return user
  }

  async findByEmail(email: string): Promise<UserResponse | null> {
    const user = await prisma.user.findUnique({
      where: { email },
      select: {
        id: true,
        name: true,
        email: true,
        createdAt: true
      }
    })
    return user
  }

  async update(id: string, data: UpdateUserData): Promise<UserResponse> {
    const user = await prisma.user.update({
      where: { id },
      data,
      select: {
        id: true,
        name: true,
        email: true,
        createdAt: true
      }
    })
    return user
  }

  async delete(id: string): Promise<void> {
    await prisma.user.delete({
      where: { id }
    })
  }

  async findMany(page: number, limit: number): Promise<UserResponse[]> {
    const users = await prisma.user.findMany({
      skip: (page - 1) * limit,
      take: limit,
      select: {
        id: true,
        name: true,
        email: true,
        createdAt: true
      }
    })
    return users
  }
}
```

### 4. Use Case (Service)

```typescript
// src/services/user/create-user.service.ts
import { UserRepositoryInterface } from '@/repositories/interfaces/user.repository.interface'
import { CreateUserData, UserResponse } from '@/dtos/user.dto'
import { hashPassword } from '@/utils/password.utils'

export class CreateUserService {
  constructor(private userRepository: UserRepositoryInterface) {}

  async execute(data: CreateUserData): Promise<UserResponse> {
    // Verificar se o usuário já existe
    const existingUser = await this.userRepository.findByEmail(data.email)
    if (existingUser) {
      throw new Error('User already exists with this email')
    }

    // Hash da senha
    const hashedPassword = await hashPassword(data.password)

    // Criar usuário
    const user = await this.userRepository.create({
      ...data,
      password: hashedPassword
    })

    return user
  }
}
```

### 5. Factory

```typescript
// src/factories/make-user.factory.ts
import { UserPrismaRepository } from '@/repositories/prisma/user.prisma.repository'
import { CreateUserService } from '@/services/user/create-user.service'
import { GetUserService } from '@/services/user/get-user.service'

export function makeCreateUserService() {
  const userRepository = new UserPrismaRepository()
  const createUserService = new CreateUserService(userRepository)
  return createUserService
}

export function makeGetUserService() {
  const userRepository = new UserPrismaRepository()
  const getUserService = new GetUserService(userRepository)
  return getUserService
}
```

### 6. Controller

```typescript
// src/controllers/user.controller.ts
import { FastifyRequest, FastifyReply } from 'fastify'
import { CreateUserDTO } from '@/dtos/user.dto'
import { makeCreateUserService } from '@/factories/make-user.factory'

export async function createUser(request: FastifyRequest, reply: FastifyReply) {
  try {
    const data = CreateUserDTO.parse(request.body)
    const createUserService = makeCreateUserService()
    
    const user = await createUserService.execute(data)
    
    return reply.status(201).send({
      success: true,
      data: user
    })
  } catch (error) {
    if (error instanceof Error) {
      return reply.status(400).send({
        success: false,
        message: error.message
      })
    }
    
    return reply.status(500).send({
      success: false,
      message: 'Internal server error'
    })
  }
}
```

## 📖 Documentação da API (Swagger)

### Configuração do Swagger

```typescript
// src/config/swagger.ts
export const swaggerOptions = {
  swagger: {
    info: {
      title: 'Biblioteca API',
      description: 'API REST para gerenciamento de biblioteca',
      version: '1.0.0'
    },
    host: 'localhost:3000',
    schemes: ['http', 'https'],
    consumes: ['application/json'],
    produces: ['application/json'],
    securityDefinitions: {
      Bearer: {
        type: 'apiKey',
        name: 'Authorization',
        in: 'header',
        description: 'JWT Authorization header using the Bearer scheme'
      }
    }
  }
}

export const swaggerUiOptions = {
  routePrefix: '/docs',
  uiConfig: {
    docExpansion: 'list',
    deepLinking: false
  },
  staticCSP: true,
  transformStaticCSP: (header: string) => header
}
```

### Rotas com Schema

```typescript
// src/routes/user.routes.ts
import { FastifyInstance } from 'fastify'
import { createUser, getUser, updateUser } from '@/controllers/user.controller'

export async function userRoutes(fastify: FastifyInstance) {
  fastify.post('/users', {
    schema: {
      description: 'Criar um novo usuário',
      tags: ['Users'],
      body: {
        type: 'object',
        properties: {
          name: { type: 'string', minLength: 2 },
          email: { type: 'string', format: 'email' },
          password: { type: 'string', minLength: 6 }
        },
        required: ['name', 'email', 'password']
      },
      response: {
        201: {
          description: 'Usuário criado com sucesso',
          type: 'object',
          properties: {
            success: { type: 'boolean' },
            data: {
              type: 'object',
              properties: {
                id: { type: 'string' },
                name: { type: 'string' },
                email: { type: 'string' },
                createdAt: { type: 'string', format: 'date-time' }
              }
            }
          }
        },
        400: {
          description: 'Dados inválidos',
          type: 'object',
          properties: {
            success: { type: 'boolean' },
            message: { type: 'string' }
          }
        }
      }
    }
  }, createUser)

  fastify.get('/users/:id', {
    schema: {
      description: 'Buscar usuário por ID',
      tags: ['Users'],
      params: {
        type: 'object',
        properties: {
          id: { type: 'string' }
        },
        required: ['id']
      },
      response: {
        200: {
          description: 'Usuário encontrado',
          type: 'object',
          properties: {
            success: { type: 'boolean' },
            data: {
              type: 'object',
              properties: {
                id: { type: 'string' },
                name: { type: 'string' },
                email: { type: 'string' },
                createdAt: { type: 'string', format: 'date-time' }
              }
            }
          }
        }
      }
    }
  }, getUser)
}
```

## 🧪 Testes

### Configuração do Vitest

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { resolve } from 'path'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node'
  },
  resolve: {
    alias: {
      '@': resolve(__dirname, './src')
    }
  }
})
```

### Exemplo de Teste Unitário

```typescript
// src/tests/unit/create-user.service.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { CreateUserService } from '@/services/user/create-user.service'
import { UserMemoryRepository } from '@/repositories/memory/user.memory.repository'

describe('CreateUserService', () => {
  let userRepository: UserMemoryRepository
  let createUserService: CreateUserService

  beforeEach(() => {
    userRepository = new UserMemoryRepository()
    createUserService = new CreateUserService(userRepository)
  })

  it('should create a new user', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      password: '123456'
    }

    const user = await createUserService.execute(userData)

    expect(user).toEqual(
      expect.objectContaining({
        name: 'John Doe',
        email: 'john@example.com'
      })
    )
    expect(user.password).toBeUndefined()
  })

  it('should not create user with existing email', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      password: '123456'
    }

    await createUserService.execute(userData)

    await expect(
      createUserService.execute(userData)
    ).rejects.toThrow('User already exists with this email')
  })
})
```

## 🚀 Scripts Úteis

### Comandos Docker

```bash
# Iniciar container do banco
docker start biblioteca-api-mysql

# Parar container
docker stop biblioteca-api-mysql

# Ver logs do container
docker logs biblioteca-api-mysql

# Remover container
docker rm biblioteca-api-mysql
```

### Comandos Prisma

```bash
# Gerar cliente Prisma
npm run db:generate

# Criar e aplicar migração
npm run db:migrate

# Abrir Prisma Studio
npm run db:studio

# Reset do banco de dados
npx prisma migrate reset

# Deploy de migrações (produção)
npx prisma migrate deploy
```

### Comandos de Desenvolvimento

```bash
# Desenvolvimento com hot reload
npm run dev

# Executar testes
npm run test

# Executar testes com interface
npm run test:ui

# Build para produção
npm run build

# Iniciar em produção
npm start
```

## ✅ Sequência de Desenvolvimento SOLID

1. **DTO** - Definir estruturas de dados
2. **Repository Interface** - Definir contratos
3. **Use Case/Service** - Implementar regras de negócio
4. **Testes** - Criar testes unitários
5. **Repository Implementation** - Implementar persistência
6. **Factory** - Gerenciar dependências
7. **Controller** - Implementar camada de apresentação
8. **Routes** - Configurar endpoints

Esta estrutura garante código limpo, testável e manutenível seguindo os princípios SOLID.