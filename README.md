# Projeto Fastify com TypeScript, Zod e Swagger

### 1. Inicializar o projeto

```bash
npm init -y
```

# Instalar dependências principais
```bash
npm install fastify zod @fastify/swagger @fastify/swagger-ui @prisma/client dotenv
```

# Instalar dependências de desenvolvimento
```bash
npm install typescript @types/node tsx prisma -D
```

# Conexão com o Docker
```bash
-> docker run --name biblioteca-api-solid-mysql  -e MARIADB_ROOT_PASSWORD=docker -e MARIADB_DATABASE=biblioteca-api -e MARIADB_USER=docker -e MARIADB_PASSWORD=docker -p 3306:3306 -d bitnami/mariadb:latest
```
# Depois eu conecto nessa url
```bash
->mysql://docker:docker@biblioteca-api-solid-mysql:3306/biblioteca-api
```

# package.json
```bash
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
```
# Configuração do TypeScript
```bash
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
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

#Código do Servidor
```bash
import fastify from 'fastify'
import { z } from 'zod'

const server = fastify({ logger: true })

// Registrar Swagger
await server.register(import('@fastify/swagger'), {
  swagger: {
    info: {
      title: 'My API',
      description: 'API documentation',
      version: '1.0.0'
    },
    host: 'localhost:3000',
    schemes: ['http'],
    consumes: ['application/json'],
    produces: ['application/json']
  }
})

await server.register(import('@fastify/swagger-ui'), {
  routePrefix: '/docs',
  uiConfig: {
    docExpansion: 'full',
    deepLinking: false
  }
})

// Schema de exemplo com Zod
const UserSchema = z.object({
  name: z.string(),
  email: z.string().email()
})

// Rota de exemplo
server.get('/health', {
  schema: {
    description: 'Health check endpoint',
    tags: ['health'],
    response: {
      200: {
        type: 'object',
        properties: {
          status: { type: 'string' },
          timestamp: { type: 'string' }
        }
      }
    }
  }
}, async (request, reply) => {
  return {
    status: 'ok',
    timestamp: new Date().toISOString()
  }
})

// Iniciar servidor
const start = async () => {
  try {
    await server.listen({ port: 3000, host: '0.0.0.0' })
    console.log('🚀 Server running on http://localhost:3000')
    console.log('📚 Docs available at http://localhost:3000/docs')
  } catch (err) {
    server.log.error(err)
    process.exit(1)
  }
}

start()
```

# Fetch from next
```bash
// pages/users.tsx
export async function getServerSideProps() {
  const res = await fetch('https://api.exemplo.com/users')
  const data = await res.json()

  return {
    props: {
      users: data,
    },
  }
}

export default function UsersPage({ users }: { users: any[] }) {
  return (
    <div>
      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  )
}
```

# Sequencia a ser seguida SOLID
```bash
Use-case -> Test (vitest) -> factory (error) -> repository(repository interface/ memory from test and prisma) -> controllers -> DTO
```

# Gerar o cliente Prisma
```bash
npx prisma generate
```
# Aplicar migrações
```bash
npx prisma migrate dev
```
# Visualizar o banco de dados com Prisma Studio
```bash
npx prisma studio
```
