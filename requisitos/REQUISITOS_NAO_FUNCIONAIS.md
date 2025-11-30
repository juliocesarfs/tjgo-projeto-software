


# Requisitos Não Funcionais Específicos (ISO/IEC 25010)

Os requisitos abaixo estão organizados conforme as características de qualidade do modelo ISO/IEC 25010, considerando acesso multiplataforma (Web, Mobile), notificações em tempo real e experiência consistente.


## 1. Desempenho e Eficiência

**NF1.1 — Tempo de Resposta**
- O sistema (API, Web e Mobile) deve responder a 95% das requisições em até 5 segundos.
- Critério: Testes de carga automatizados.

**NF1.2 — Autenticação Rápida**
- O tempo de resposta para autenticação de usuário (login) não deve exceder 2 segundo em 80% dos casos.
- Critério: Logs de autenticação e testes automatizados.

**NF1.3 — Suporte a Usuários Simultâneos**
- O sistema deve suportar pelo menos 500 usuários autenticados simultaneamente em múltiplos canais.
- Critério: Testes de estresse.

**NF1.4 — Sincronização Multiplataforma**
- Sincronização entre canais em até 5 segundos.
- Critério: Testes de sincronização e notificações.

## 2. Segurança

**NF2.1 — Comunicação Segura**
- Todas as comunicações entre cliente e servidor devem ser feitas via HTTPS, com TLS 1.2 ou superior.
- Critério: Testes de segurança e análise de headers devem comprovar a ausência de conexões HTTP.

**NF2.2 — Armazenamento Seguro de Senhas**
- As senhas dos usuários devem ser armazenadas usando hash seguro (bcrypt, salt >= 10).
- Critério: Auditoria do banco de dados e código-fonte.

**NF2.3 — Autenticação JWT**
- O sistema deve implementar autenticação JWT para todas as rotas protegidas, com tokens expirando após 24 horas.
- Critério: Testes de API e análise de configuração.

**NF2.4 — Proteção contra Ataques Comuns**
- O sistema deve ser protegido contra ataques de injeção SQL, XSS, CSRF e brute force.
- Critério: Testes de segurança automatizados e revisão de código.

## 3. Confiabilidade

**NF3.1 — Disponibilidade**
- Disponibilidade de 99,5% durante horário comercial, exceto manutenções.
- Critério: Monitoramento de uptime.

**NF3.2 — Backup Diário**
- Backups semanais do banco de dados em local seguro.
- Critério: Logs de backup e testes de restauração.

**NF3.3 — Recuperação Rápida**
- Deve ser possível restaurar o sistema a partir de um backup em até 2 horas após incidente crítico.
- Critério: Testes de recuperação documentados.


## 4. Usabilidade

**NF4.1 — Interface Responsiva e Consistente**
- A interface deve ser responsiva, acessível e manter experiência consistente entre web e mobile, adaptando-se a diferentes tamanhos de tela e plataformas.
- Critério: Testes em navegadores, dispositivos móveis e validação de padrões de design.

**NF4.2 — Mensagens de Erro Claras**
- Mensagens de erro devem ser claras, orientativas e, quando possível, sugerir ações corretivas ao usuário em todos os canais.
- Critério: Revisão de textos e testes de usabilidade.

**NF4.3 — Notificações em Tempo Real**
- O sistema deve enviar notificações em tempo real para web e mobile sempre que houver eventos relevantes (ex: nova tarefa, comentário, atribuição).
- Critério: Testes de recebimento de notificações push/web.


## 5. Compatibilidade

**NF5.1 — Navegadores e Plataformas Suportadas**
- O sistema deve ser compatível com as versões mais recentes dos navegadores Chrome, Firefox e Edge, além de Android (versão 8+) e iOS (versão 13+).
- Critério: Testes de compatibilidade em múltiplos navegadores e dispositivos móveis.

## 6. Manutenibilidade

**NF6.1 — Código Documentado**
- O código-fonte deve ser documentado, seguir boas práticas de desenvolvimento e padrões definidos pelo time.
- Critério: Revisão de código e ferramentas de análise estática.

**NF6.2 — Testes Automatizados**
- O sistema deve possuir cobertura de testes automatizados (unitários e de integração) de pelo menos 70% das funcionalidades críticas.
- Critério: Relatórios de cobertura de testes.

## 7. Portabilidade

**NF7.1 — Ambientes de Implantação**
- O sistema deve ser implantável em ambientes Windows e Linux sem necessidade de grandes adaptações.
- Critério: Testes de implantação em ambos os ambientes.

---

## 8. Detalhes de Implementação Técnica

### 8.1 Segurança

#### RNF-01: Autenticação baseada em Token
- **Tipo**: JWT (JSON Web Tokens)
- **Secret**: Hardcoded (DEVE ser movido para variáveis de ambiente)
- **Valor atual**: '8b6c-fef4-4d7d-b8ab-74kl-3729'
- **Locais**: `src/infra/notes-system.module.ts:32` e `src/infra/auth/auth.module.ts:19`

#### RNF-02: Guard de autenticação
- **Implementação**: Middleware de validação JWT
- **Funcionalidade**:
  - Validar presença do header `Authorization: Bearer <token>`
  - Verificar assinatura do token
  - Descodificar e validar conteúdo
  - Anexar informações do usuário ao request object

#### RNF-03: CORS (Cross-Origin Resource Sharing)
- **Configuração atual**: Permissiva para desenvolvimento
- **Origem**: Aceita qualquer origem
- **Métodos**: GET, POST, PUT, DELETE, OPTIONS
- **Headers**: Content-Type, Authorization
- **Credenciais**: Habilitadas
- **Recomendação**: Restringir para produção

#### RNF-04: Validação de entrada
- **Framework**: class-validator com decorators
- **Aplicação**: Todos os DTOs validam campos antes de usar
- **Tipagem**: class-transformer para transformação de tipos

#### RNF-05: Segurança de dados sensíveis
- **Senhas**: Nunca retornar em respostas
- **Tokens**: Retornar apenas após autenticação bem-sucedida
- **Dados pessoais**: Respeitar permissões de acesso

### 8.2 Performance e Escalabilidade

#### RNF-06: Indexação de banco de dados
- **Índices implementados**:
  - User.email (busca por email)
  - User.teamId (filtros por time)
  - Note.createdById (filtros por criador)
  - Note.assignedToId (filtros por responsável)
  - Note.teamId (filtros por time)
  - Note.status (filtros por status)
  - Note.category (filtros por categoria)

### 8.3 Arquitetura e Padrões

#### RNF-07: Clean Architecture
- **Camadas**: Application, Infrastructure
- **Separação**: Lógica de negócio isolada de frameworks

#### RNF-08: Repository Pattern
- **Implementação**: `PrismaUserRepository`, `PrismaNoteRepository`
- **Abstração**: Operações de banco de dados isoladas
- **Flexibilidade**: Trocar implementação sem afetar lógica

#### RNF-09: Use Case Pattern
- **Estrutura**: Cada operação em classe dedicada com método `execute()`
- **Responsabilidade única**: Um caso de uso, uma operação
- **Testabilidade**: Casos de uso testáveis independentemente

#### RNF-10: Dependency Injection
- **Framework**: NestJS built-in DI container
- **Benefit**: Módulos e dependências declaradas explicitamente

#### RNF-11: DTOs (Data Transfer Objects)
- **Uso**: Validação e transformação de dados de entrada
- **Bibliotecas**: class-validator, class-transformer
- **Aplicação**: Separação entre entrada e lógica

### 8.4 Tratamento de Erros

#### RNF-12: Respostas padronizadas
- **Formato**:
  ```json
  {
    "success": boolean,
    "message": string,
    "data": any
  }
  ```
- **Uso**: Todos os endpoints devem retornar formato consistente

#### RNF-13: Códigos de status HTTP
- **201**: Criação bem-sucedida
- **200**: Operação bem-sucedida
- **400**: Erro de validação / requisição inválida
- **401**: Não autenticado
- **403**: Não autorizado
- **404**: Recurso não encontrado
- **500**: Erro interno do servidor

#### RNF-14: Tratamento de exceções
- **HttpException**: Lançadas em controllers para erros específicos
- **Try-catch**: Implementado em todos os endpoints
- **Mensagens**: Informativos úteis para clientes

### 8.5 Banco de Dados

#### RNF-15: ORM Prisma
- **Versão**: 6.16
- **Provider**: PostgreSQL
- **Benefícios**:
  - Type-safe queries
  - Migrations automáticas
  - Schema versioning

#### RNF-16: Relacionamentos
- **One-to-Many**: User → Notes (criadas e atribuídas)
- **One-to-One**: Team → Admin User
- **Many-to-Many**: User ↔ Team (membros)

#### RNF-17: Timestamp automático
- **created_at**: Preenchido automaticamente na criação
- **updated_at**: Atualizado automaticamente em modificações
