# 🚀 Sistema de Microserviços — Gestão de Usuários

## Arquitetura

```
                        ┌─────────────────────────────────┐
                        │         API Gateway :3000        │
                        │   (CORS · Proxy · Swagger Docs) │
                        └──────────┬──────────┬───────────┘
                                   │          │
                    ┌──────────────▼──┐   ┌───▼─────────────┐
                    │  Auth Service   │   │  User Service    │
                    │    :3001        │   │    :3002         │
                    │  JWT · Login    │   │  CRUD Usuários   │
                    └──────────┬──────┘   └───────┬─────────┘
                               │                  │
                    ┌──────────▼──────────────────▼─────────┐
                    │          PostgreSQL :5432              │
                    │    users · refresh_tokens              │
                    └───────────────────────────────────────┘
```

## Pré-requisitos
- [Docker](https://www.docker.com/) + Docker Compose

## Como executar

```bash
# Subir todos os serviços
docker compose up --build

# Rodar em background
docker compose up --build -d

# Parar
docker compose down

# Limpar tudo (incluindo volume do banco)
docker compose down -v
```

## Documentação dos endpoints

| Serviço       | Swagger UI                              |
|---------------|-----------------------------------------|
| **Gateway**   | http://localhost:3000/api-docs          |
| Auth Service  | http://localhost:3001/api-docs          |
| User Service  | http://localhost:3002/api-docs          |

---

## Endpoints

### 🔐 Autenticação (`/api/auth`)

| Método | Rota                    | Descrição                       | Auth |
|--------|-------------------------|---------------------------------|------|
| POST   | `/api/auth/login`       | Login com e-mail e senha        | ❌   |
| POST   | `/api/auth/refresh`     | Renova o access token           | ❌   |
| POST   | `/api/auth/logout`      | Logout — invalida refresh token | ✅   |
| POST   | `/api/auth/validate`    | Valida um token JWT             | ❌   |

### 👤 Usuários (`/api/users`)

| Método | Rota              | Descrição                       | Auth   | Role  |
|--------|-------------------|---------------------------------|--------|-------|
| POST   | `/api/users`      | Cria um usuário (cadastro)      | ❌     | —     |
| GET    | `/api/users`      | Lista todos os usuários         | ✅     | admin |
| GET    | `/api/users/me`   | Perfil do usuário logado        | ✅     | any   |
| GET    | `/api/users/:id`  | Consulta usuário por ID         | ✅     | any*  |
| PUT    | `/api/users/:id`  | Atualiza usuário                | ✅     | any*  |
| DELETE | `/api/users/:id`  | Remove usuário                  | ✅     | admin |

> *Usuário comum só pode acessar/editar o próprio perfil

---

## Exemplos de uso

### 1. Criar conta
```bash
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "João Silva",
    "email": "joao@email.com",
    "password": "Senha@123"
  }'
```

### 2. Login
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@sistema.com",
    "password": "Admin@123"
  }'
# Retorna: accessToken e refreshToken
```

### 3. Consultar perfil
```bash
curl http://localhost:3000/api/users/me \
  -H "Authorization: Bearer SEU_ACCESS_TOKEN"
```

### 4. Listar usuários (admin)
```bash
curl "http://localhost:3000/api/users?page=1&limit=10&search=joao" \
  -H "Authorization: Bearer TOKEN_DO_ADMIN"
```

### 5. Renovar token
```bash
curl -X POST http://localhost:3000/api/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{ "refreshToken": "SEU_REFRESH_TOKEN" }'
```

---

## Credencial padrão (admin)
```
E-mail: admin@sistema.com
Senha:  Admin@123
```

## Variáveis de ambiente

| Variável          | Padrão                | Descrição              |
|-------------------|-----------------------|------------------------|
| `JWT_SECRET`      | —                     | Chave secreta JWT      |
| `JWT_EXPIRES_IN`  | `1h`                  | Expiração do token     |
| `DATABASE_URL`    | —                     | String de conexão PG   |
| `PORT`            | varia por serviço     | Porta do serviço       |

## Segurança implementada
- ✅ Senhas com hash **bcrypt** (salt rounds: 10)
- ✅ Autenticação via **JWT** (access + refresh token)
- ✅ Refresh tokens armazenados no banco com expiração
- ✅ Controle de acesso por **role** (admin / user)
- ✅ **CORS** configurado em todos os serviços
- ✅ Usuário não pode elevar a própria role
