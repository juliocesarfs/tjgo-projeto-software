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

### 1.6 Notificações de Email

#### RF-22: Criar registro de notificação de email
- **Descrição**: Registrar notificações de email enviadas aos usuários
- **Dados da notificação**:
  - ID da notificação
  - ID do usuário destinatário
  - ID da nota relacionada (opcional)
  - Tipo de notificação (task_assigned, task_due, comment_added, team_invite, etc.)
  - Mensagem da notificação
  - Data de envio
  - Data de leitura (null se não lida)
- **Validações**:
  - Usuário destinatário deve existir
  - Tipo de notificação deve ser válido
- **Saída**: Notificação criada
- **Processo**: Automático quando emails são enviados

#### RF-23: Listar notificações de email do usuário
- **Descrição**: Recuperar todas as notificações de email de um usuário
- **Entrada**: ID do usuário
- **Parâmetros opcionais**:
  - `type`: filtrar por tipo de notificação
  - `read`: filtrar por lidas (true) ou não lidas (false)
  - `startDate`: data inicial
  - `endDate`: data final
- **Saída**: Array de notificações com todos os dados
- **Endpoint**: `GET /notifications/user/:userId`
- **Autenticação**: Requerida

#### RF-24: Marcar notificação como lida
- **Descrição**: Registrar que usuário visualizou a notificação
- **Entrada**: ID da notificação
- **Processo**: Atualizar campo `readAt` com timestamp atual
- **Validações**:
  - Notificação deve existir
  - Usuário deve ser o destinatário
- **Saída**: Notificação atualizada
- **Endpoint**: `PUT /notifications/:id/read`
- **Autenticação**: Requerida

#### RF-25: Filtrar notificações por tipo ou status
- **Descrição**: Permitir filtragem específica de notificações
- **Tipos de notificação**:
  - `task_assigned`: Tarefa atribuída
  - `task_due`: Tarefa com prazo próximo/vencida
  - `comment_added`: Comentário adicionado
  - `team_invite`: Convite para time
  - `task_shared`: Tarefa compartilhada
- **Status**:
  - `unread`: não lidas (readAt = null)
  - `read`: lidas (readAt ≠ null)
- **Endpoint**: Via query parameters em `GET /notifications/user/:userId`
- **Autenticação**: Requerida

#### RF-26: Enviar email para o usuário
- **Descrição**: Manter funcionalidade existente de envio de emails
- **Processo**:
  1. Criar registro da notificação (RF-22)
  2. Enviar email via serviço externo (SendGrid/AWS SES)
  3. Usuário pode visualizar histórico posteriormente (RF-23)
- **Eventos que disparam emails**:
  - Atribuição de tarefa
  - Tarefa vencida
  - Comentário adicionado
  - Convite para time
  - Tarefa compartilhada
- **Autenticação**: Sistema interno

---

