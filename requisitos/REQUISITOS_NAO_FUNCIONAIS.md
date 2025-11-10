
# Requisitos Não Funcionais (ISO/IEC 25010)

Os requisitos abaixo estão organizados conforme as características de qualidade do modelo ISO/IEC 25010.

## 1. Desempenho e Eficiência
- O sistema deve responder a qualquer requisição em até 2 segundos para 95% das operações.
- O tempo de autenticação não deve exceder 1 segundo.
- O sistema deve suportar pelo menos 500 usuários simultâneos sem degradação perceptível de desempenho.

## 2. Segurança
- Todas as comunicações devem ser feitas via HTTPS.
- As senhas dos usuários devem ser armazenadas usando hash seguro (bcrypt, salt >= 10).
- O sistema deve implementar autenticação JWT para todas as rotas protegidas.
- Tokens de acesso devem expirar após 24 horas.

## 3. Confiabilidade
- O sistema deve estar disponível 99,5% do tempo durante o horário comercial (8h-18h).
- Backups do banco de dados devem ser realizados diariamente.
- Deve ser possível restaurar o sistema a partir de um backup em até 2 horas.

## 4. Usabilidade
- A interface deve ser responsiva e acessível em dispositivos móveis e desktop.
- Mensagens de erro devem ser claras e orientativas.

## 5. Compatibilidade
- O sistema deve ser compatível com os navegadores Chrome, Firefox e Edge nas versões mais recentes.

## 6. Manutenibilidade
- O código-fonte deve ser documentado e seguir boas práticas de desenvolvimento para facilitar manutenção e evolução.

## 7. Portabilidade
- O sistema deve ser implantável em ambientes Windows e Linux sem necessidade de grandes adaptações.
