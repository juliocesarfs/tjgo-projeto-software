# Requisitos do Projeto - Control Tasks Backend

## 1. Requisitos Funcionais

### 1.1 Gerenciamento de Usuários

#### RF-01: Criar novo usuário
- **Descrição**: Sistema deve permitir criar novos usuários na plataforma
- **Entrada**: Email, nome, senha
- **Validações**:
  - Email válido e único
  - Nome: mínimo 2, máximo 100 caracteres
  - Senha: mínimo 6 caracteres
  - Tipo de usuário: ADMIN ou USER (padrão: USER)
- **Saída**: Usuário criado com ID, email, nome e tipo
- **Endpoint**: `POST /users`
- **Autenticação**: Não requerida

#### RF-02: Autenticar usuário
- **Descrição**: Gerar token API para acesso aos endpoints protegidos
- **Entrada**: Email e senha
- **Processo**:
  - Validar credenciais (verificar email e senha com bcrypt)
  - Gerar JWT token com 365 dias de validade
  - Armazenar token na tabela ApiToken
  - Revogar token anterior do usuário (se existir)
- **Saída**: Token de autenticação, dados do usuário (id, email, nome, tipo)
- **Endpoint**: `POST /users/authenticate`
- **Autenticação**: Não requerida

#### RF-03: Obter dados do usuário
- **Descrição**: Recuperar informações de um usuário específico
- **Entrada**: ID do usuário (via parâmetro de rota)
- **Saída**: Dados do usuário (id, email, nome, tipo de usuário, time)
- **Endpoint**: `GET /users/:id`
- **Autenticação**: Requerida (ApiTokenGuard)

#### RF-04: Atualizar perfil do usuário
- **Descrição**: Usuários podem atualizar seu próprio perfil; ADMINs podem atualizar qualquer perfil
- **Entrada**: ID do usuário, nome (opcional), email (opcional), senha (opcional)
- **Validações**:
  - Usuário comum pode atualizar apenas seu próprio perfil
  - ADMIN pode atualizar qualquer perfil
  - Nome: mínimo 2, máximo 100 caracteres (se fornecido)
  - Email: válido e único (se fornecido)
  - Senha: mínimo 6 caracteres (se fornecido)
- **Saída**: Dados do usuário atualizado
- **Endpoint**: `PUT /users/:id`
- **Autenticação**: Requerida (ApiTokenGuard)

---

### 1.2 Gerenciamento de Notas/Tarefas

#### RF-05: Criar nova nota/tarefa
- **Descrição**: Criar uma nota ou tarefa no sistema
- **Entrada**:
  - Título (obrigatório): string
  - Descrição (opcional): string
  - Categoria (opcional): "note" ou "task"
  - Prioridade (opcional): LOW, MEDIUM (padrão), HIGH, URGENT
  - Status (opcional): PENDING (padrão), TODO, IN_PROGRESS, REVIEW, COMPLETED, CANCELLED
  - Criador (obrigatório): ID do usuário
  - Atribuído a (opcional): ID do usuário responsável
  - Time (opcional): ID do time
  - Data de vencimento (opcional): data em formato ISO
- **Validações**:
  - Título é obrigatório
  - Criador deve ser um usuário válido
- **Saída**: Nota criada com todos os dados
- **Endpoint**: `POST /notes`
- **Autenticação**: Requerida

#### RF-06: Listar todas as notas com filtros
- **Descrição**: Recuperar todas as notas do sistema com filtros opcionais
- **Parâmetros de filtro**:
  - `title`: busca parcial no título
  - `status`: PENDING, TODO, IN_PROGRESS, REVIEW, COMPLETED, CANCELLED
  - `priority`: LOW, MEDIUM, HIGH, URGENT
  - `category`: note, task
  - `assignedToId`: ID do usuário atribuído
  - `teamId`: ID do time
  - `createdById`: ID do criador
  - `startDate`: data inicial (ISO)
  - `endDate`: data final (ISO)
- **Saída**: Array de notas que correspondem aos filtros
- **Endpoint**: `GET /notes`
- **Autenticação**: Requerida

#### RF-07: Listar notas do usuário
- **Descrição**: Recuperar notas criadas ou atribuídas ao usuário
- **Entrada**: ID do usuário e filtros (mesmos de RF-06)
- **Lógica**: Retorna notas onde `createdById = userId` OR `assignedToId = userId`
- **Saída**: Array de notas do usuário
- **Endpoint**: `GET /notes/my-notes/:userId`
- **Autenticação**: Requerida

#### RF-08: Obter estatísticas do usuário
- **Descrição**: Recuperar métricas de produtividade do usuário
- **Entrada**: ID do usuário
- **Saída**: Estatísticas (número de notas por status, total de notas, etc.)
- **Endpoint**: `GET /notes/stats/:userId`
- **Autenticação**: Requerida

#### RF-09: Obter nota específica
- **Descrição**: Recuperar detalhes de uma única nota
- **Entrada**: ID da nota
- **Saída**: Dados completos da nota
- **Endpoint**: `GET /notes/:id`
- **Autenticação**: Requerida

#### RF-10: Atualizar nota
- **Descrição**: Modificar propriedades de uma nota existente
- **Entrada**: ID da nota e campos a atualizar
  - Título (opcional): 1-40 caracteres
  - Descrição (opcional): máximo 300 caracteres
  - Status (opcional): PENDING, TODO, IN_PROGRESS, REVIEW, COMPLETED, CANCELLED
  - Prioridade (opcional): LOW, MEDIUM, HIGH, URGENT
  - Categoria (opcional): note, task
  - Data de vencimento (opcional): data ISO
- **Saída**: Nota atualizada
- **Endpoint**: `PUT /notes/:id`
- **Autenticação**: Requerida

#### RF-11: Deletar nota
- **Descrição**: Remover uma nota permanentemente do sistema
- **Entrada**: ID da nota
- **Saída**: Mensagem de sucesso
- **Endpoint**: `DELETE /notes/:id`
- **Autenticação**: Requerida

#### RF-12: Atribuir nota a usuário
- **Descrição**: Designar uma nota a um usuário responsável
- **Entrada**: ID da nota, ID do usuário
- **Saída**: Nota com atribuição atualizada
- **Endpoint**: `PUT /notes/:id/assign`
- **Autenticação**: Requerida

#### RF-13: Remover atribuição de nota
- **Descrição**: Remover responsável de uma nota
- **Entrada**: ID da nota
- **Saída**: Nota sem atribuição
- **Endpoint**: `PUT /notes/:id/unassign`
- **Autenticação**: Requerida

---

### 1.3 Lógica de Domínio - Entidade Nota

#### RF-14: Transições de estado da nota
- **Descrição**: Nota deve suportar transições de estado de forma controlada
- **Estados disponíveis**: PENDING, TODO, IN_PROGRESS, REVIEW, COMPLETED, CANCELLED
- **Métodos de transição**:
  - `complete()`: PENDING/TODO/IN_PROGRESS/REVIEW → COMPLETED
  - `cancel()`: qualquer estado → CANCELLED
  - `startProgress()`: qualquer estado → IN_PROGRESS
  - `putTodo()`: qualquer estado → TODO
  - `sendToReview()`: qualquer estado → REVIEW
- **Efeito colateral**: Atualizar `updated_at` ao mudar estado

#### RF-15: Verificar se nota está vencida
- **Descrição**: Sistema deve identificar notas vencidas
- **Lógica**: Nota está vencida se `dueDate` < data atual E status ≠ COMPLETED
- **Uso**: Para alertas e filtros

#### RF-16: Verificações de permissão em nota
- **Descrição**: Validar se usuário tem permissão para editar nota
- **Regras**:
  - Criador sempre pode editar sua nota
  - ADMIN do time pode editar notas do time
  - Outros usuários não podem editar
- **Método**: `canBeEditedBy(userId, isTeamAdmin)`

#### RF-17: Atribuição de nota
- **Descrição**: Gerenciar responsável pela nota
- **Métodos**:
  - `assignTo(userId)`: Designar usuário
  - `unassign()`: Remover responsável
  - `isAssignedTo(userId)`: Verificar se atribuído
- **Efeito colateral**: Atualizar timestamp ao atribuir/desatribuir

---

### 1.4 Colaboração em Equipe

#### RF-18: Time com administrador
- **Descrição**: Criar estrutura de times com gerenciador
- **Dados do time**:
  - Nome
  - Código único (identificador)
  - Descrição (opcional)
  - ID do administrador
  - Membros (lista de usuários)
- **Relacionamentos**:
  - Um time pertence a um único administrador
  - Um time pode ter múltiplos membros
  - Notas podem ser associadas a um time
  - Usuários podem pertencer a um time

#### RF-19: Notas compartilhadas em time
- **Descrição**: Notas podem ser criadas no contexto de um time
- **Lógica**:
  - Notas com `teamId` são compartilhadas entre membros
  - ADMIN do time pode editar qualquer nota do time
  - Membros podem visualizar notas do time

---

### 1.5 Autenticação e Tokens

#### RF-20: Geração de API Tokens
- **Descrição**: Criar tokens JWT para autenticação em requisições
- **Dados armazenados**:
  - Token único
  - ID do usuário
  - Nome/descrição do token
  - Data de expiração (opcional)
  - Flag de revogação
- **Validação**:
  - Verificar assinatura JWT
  - Verificar expiração (se aplicável)
  - Verificar flag de revogação

#### RF-21: Revogação automática de token anterior
- **Descrição**: Quando usuário faz login, seu token anterior é revogado
- **Lógica**: Apenas um token ativo por usuário por vez
- **Exceção**: Tokens sem expiração podem ser reutilizados

---

## 2. Requisitos Não Funcionais

### 2.1 Segurança

#### RNF-01: Hash de senhas
- **Implementação**: bcrypt com salt rounds = 10
- **Aplicação**: Todas as senhas devem ser hash antes de armazenar
- **Verificação**: Comparação segura de hash na autenticação

#### RNF-02: Autenticação baseada em Token
- **Tipo**: JWT (JSON Web Tokens)
- **Secret**: Hardcoded (DEVE ser movido para variáveis de ambiente)
- **Valor atual**: '8b6c-fef4-4d7d-b8ab-74kl-3729'
- **Locais**: `src/infra/notes-system.module.ts:32` e `src/infra/auth/auth.module.ts:19`

#### RNF-03: Guard de autenticação
- **Implementação**: `ApiTokenGuard`
- **Funcionalidade**:
  - Validar presença do header `Authorization: Bearer <token>`
  - Verificar assinatura do token
  - Descodificar e validar conteúdo
  - Anexar informações do usuário ao request object

#### RNF-04: CORS (Cross-Origin Resource Sharing)
- **Configuração atual**: Permissiva para desenvolvimento
- **Origem**: Aceita qualquer origem
- **Métodos**: GET, POST, PUT, DELETE, OPTIONS
- **Headers**: Content-Type, Authorization
- **Credenciais**: Habilitadas
- **Recomendação**: Restringir para produção

#### RNF-05: Validação de entrada
- **Framework**: class-validator com decorators
- **Aplicação**: Todos os DTOs validam campos antes de usar
- **Tipagem**: class-transformer para transformação de tipos

#### RNF-06: Segurança de dados sensíveis
- **Senhas**: Nunca retornar em respostas
- **Tokens**: Retornar apenas após autenticação bem-sucedida
- **Dados pessoais**: Respeitar permissões de acesso

---

### 2.2 Performance e Escalabilidade

#### RNF-07: Indexação de banco de dados
- **Índices implementados**:
  - User.email (busca por email)
  - User.teamId (filtros por time)
  - Note.createdById (filtros por criador)
  - Note.assignedToId (filtros por responsável)
  - Note.teamId (filtros por time)
  - Note.status (filtros por status)
  - Note.category (filtros por categoria)
  - ApiToken.token (busca rápida de token)
  - ApiToken.userId (tokens por usuário)

#### RNF-08: Paginação (não implementada ainda)
- **Recomendação**: Adicionar paginação em endpoints de listagem
- **Parâmetros**: `limit`, `offset` ou `page`, `pageSize`

#### RNF-09: Cache (não implementado)
- **Recomendação**: Implementar cache Redis para:
  - Dados do usuário
  - Notas frequentemente consultadas
  - Estatísticas do usuário

#### RNF-10: Soft delete (estrutura preparada)
- **Status**: Estrutura preparada mas não ativa
- **Benefício**: Preservar dados históricos

---

### 2.3 Arquitetura e Padrões

#### RNF-11: Clean Architecture
- **Camadas**: Application, Infrastructure
- **Separação**: Lógica de negócio isolada de frameworks

#### RNF-12: Repository Pattern
- **Implementação**: `PrismaUserRepository`, `PrismaNoteRepository`, `PrismaApiTokenRepository`
- **Abstração**: Operações de banco de dados isoladas
- **Flexibilidade**: Trocar implementação sem afetar lógica

#### RNF-13: Use Case Pattern
- **Estrutura**: Cada operação em classe dedicada com método `execute()`
- **Responsabilidade única**: Um caso de uso, uma operação
- **Testabilidade**: Casos de uso testáveis independentemente

#### RNF-14: Dependency Injection
- **Framework**: NestJS built-in DI container
- **Benefit**: Módulos e dependências declaradas explicitamente

#### RNF-15: DTOs (Data Transfer Objects)
- **Uso**: Validação e transformação de dados de entrada
- **Bibliotecas**: class-validator, class-transformer
- **Aplicação**: Separação entre entrada e lógica

---

### 2.4 Tratamento de Erros

#### RNF-16: Respostas padronizadas
- **Formato**:
  ```json
  {
    "success": boolean,
    "message": string,
    "data": any
  }
  ```
- **Uso**: Todos os endpoints devem retornar formato consistente

#### RNF-17: Códigos de status HTTP
- **201**: Criação bem-sucedida
- **200**: Operação bem-sucedida
- **400**: Erro de validação / requisição inválida
- **401**: Não autenticado
- **403**: Não autorizado
- **404**: Recurso não encontrado
- **500**: Erro interno do servidor

#### RNF-18: Tratamento de exceções
- **HttpException**: Lançadas em controllers para erros específicos
- **Try-catch**: Implementado em todos os endpoints
- **Mensagens**: Informativos úteis para clientes

---

### 2.5 Banco de Dados

#### RNF-19: ORM Prisma
- **Versão**: 6.16
- **Provider**: PostgreSQL
- **Benefícios**:
  - Type-safe queries
  - Migrations automáticas
  - Schema versioning

#### RNF-20: Relacionamentos
- **One-to-Many**: User → Notes (criadas e atribuídas)
- **One-to-One**: Team → Admin User
- **Many-to-Many**: User ↔ Team (membros)
- **Cascade delete**: ApiToken deletado quando User deletado

#### RNF-21: Timestamp automático
- **created_at**: Preenchido automaticamente na criação
- **updated_at**: Atualizado automaticamente em modificações

---

### 2.6 Código e Qualidade

#### RNF-22: TypeScript
- **Versão**: 5.7
- **Modo**: Strict habilitado
- **Benefício**: Type safety em todo o código

#### RNF-23: Linting e Formatação
- **ESLint**: Análise estática de código
- **Prettier**: Formatação automática
- **Uso**: `npm run lint`, `npm run format`

#### RNF-24: Estrutura de arquivos
```
src/
├── application/        # Lógica de negócio pura
│   ├── entities/       # Modelos de domínio
│   └── use-cases/      # Casos de uso
├── infra/             # Infraestrutura
│   ├── controllers/    # HTTP handlers
│   ├── database/       # Prisma e repositórios
│   ├── auth/           # Autenticação
│   └── dtos/           # Validação de entrada
└── helpers/            # Utilitários
```

#### RNF-25: Testing
- **Framework**: Jest
- **Cobertura**: Preparado para testes unitários
- **Comandos**: `npm test`, `npm run test:cov`

---

### 2.7 Configuração e Deployment

#### RNF-26: Environment Variables
- **Requeridas**:
  - `DATABASE_URL`: Conexão PostgreSQL
  - `SHADOW_DATABASE_URL`: Database espelho para migrações
  - `PORT`: Porta do servidor (padrão: 3000)
  - `JWT_SECRET`: Secret para assinatura JWT
  - `JWT_EXPIRES_IN`: Tempo de expiração do token

#### RNF-27: Build Process
- **Compilação**: TypeScript → JavaScript
- **Output**: Diretório `dist/`
- **Production**: `npm run build && npm run start:prod`

#### RNF-28: Desenvolvimento
- **Hot reload**: `npm run start:dev`
- **Debugging**: `npm run start:debug` com suporte a breakpoints

---

## 3. Matriz de Rastreabilidade

| ID RF | Descrição | Endpoint | Método | Autenticação |
|-------|-----------|----------|--------|--------------|
| RF-01 | Criar usuário | /users | POST | Não |
| RF-02 | Autenticar | /users/authenticate | POST | Não |
| RF-03 | Obter usuário | /users/:id | GET | Sim |
| RF-04 | Atualizar usuário | /users/:id | PUT | Sim |
| RF-05 | Criar nota | /notes | POST | Sim |
| RF-06 | Listar notas | /notes | GET | Sim |
| RF-07 | Minhas notas | /notes/my-notes/:userId | GET | Sim |
| RF-08 | Estatísticas | /notes/stats/:userId | GET | Sim |
| RF-09 | Nota específica | /notes/:id | GET | Sim |
| RF-10 | Atualizar nota | /notes/:id | PUT | Sim |
| RF-11 | Deletar nota | /notes/:id | DELETE | Sim |
| RF-12 | Atribuir nota | /notes/:id/assign | PUT | Sim |
| RF-13 | Remover atribuição | /notes/:id/unassign | PUT | Sim |

---

## 4. Dados de Entrada - Restrições e Validações

### Usuários
| Campo | Tipo | Obrigatório | Validações |
|-------|------|-------------|-----------|
| email | String | Sim | Email válido, único |
| name | String | Sim | 2-100 caracteres |
| password | String | Sim | Mínimo 6 caracteres |
| userType | Enum | Não | ADMIN \| USER (padrão: USER) |

### Notas
| Campo | Tipo | Obrigatório | Validações |
|-------|------|-------------|-----------|
| title | String | Sim | Mínimo 1 caractere |
| description | String | Não | Máximo 300 caracteres |
| status | Enum | Não | PENDING (padrão), TODO, IN_PROGRESS, REVIEW, COMPLETED, CANCELLED |
| priority | Enum | Não | LOW, MEDIUM (padrão), HIGH, URGENT |
| category | Enum | Não | note, task |
| createdById | Number | Sim | ID de usuário válido |
| assignedToId | Number | Não | ID de usuário válido |
| teamId | Number | Não | ID de time válido |
| dueDate | DateTime | Não | Data futura (recomendado) |

---

## 5. Próximos Passos Sugeridos

1. **Segurança**: Mover JWT_SECRET para environment variables
2. **Performance**: Implementar paginação em listagens
3. **Cache**: Adicionar Redis para dados frequentes
4. **Testes**: Implementar test suite completo
5. **Logging**: Sistema de logs estruturado
6. **Rate Limiting**: Proteção contra abuso de API
7. **Documentação**: Swagger/OpenAPI
8. **Soft Delete**: Ativar soft delete para dados sensíveis
9. **Backup**: Estratégia de backup do banco de dados
10. **Monitoramento**: APM e alertas para produção
