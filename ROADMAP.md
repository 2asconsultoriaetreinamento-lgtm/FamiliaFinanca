# Roadmap - FamiliaFinanca

## Status Atual (14 de Novembro de 2025)

### ✅ Concluído

#### Fase 1: Banco de Dados (100%)
- ✅ Criadas 8 tabelas no Supabase
  - users
  - families
  - family_members
  - family_invites
  - accounts
  - bills
  - transactions
  - family_activity_log
- ✅ Índices criados
- ✅ Foreign keys com ON DELETE CASCADE
- ✅ Unique constraints configuradas
- ✅ Default values definidos

#### Fase 2: Row Level Security - RLS (100%)
- ✅ RLS habilitado em todas as 8 tabelas
- ✅ 16+ políticas de acesso criadas
  - Isolamento pessoal vs. familiar
  - Type casting (UUID → TEXT)
  - Validação de membros via EXISTS
- ✅ Teste de RLS policies validado

#### Fase 3: Repositório e Documentação (100%)
- ✅ Repositório GitHub criado (FamiliaFinanca)
- ✅ MIT License configurado
- ✅ .gitignore (Node template)
- ✅ README.md com visão geral do projeto
- ✅ 5 arquivos de documentação em docs/:
  - ARCHITECTURE.md
  - SCHEMA.md
  - SECURITY.md
  - SETUP.md
  - API.md

### 🔄 Em Progresso

#### Fase 4: Backend TypeScript (Replit) - 30%
- ✅ Estrutura do projeto criada
- ✅ Schema TypeScript com Drizzle ORM
  - UserRoleEnum (admin, member)
  - TransactionTypeEnum (income, expense)
  - BillStatusEnum (paid, pending, overdue)
  - RecurrenceTypeEnum (daily, weekly, monthly, etc.)
- ✅ Zod para validação de dados
- ⚠️ **Problema Identificado**: Credenciais PostgreSQL Neon inválidas/expiradas
  - Solução temporária: MemStorage (em memória)
  - Próximo passo: Reconectar Supabase (melhor que Neon)

### 📋 Próximos Passos Planejados

## Fase 4: Backend API (Semana 1)

### 4.1 - Resolver Banco de Dados (1-2 horas)
**Status**: Crítico
**Ação**:
```
1. Opção A (Recomendado): Usar Supabase ao invés de Neon
   - Já temos projeto Supabase criado
   - Integrar DATABASE_URL do Supabase no Replit
   - Remover dependência do Neon

2. Opção B: Corrigir credenciais Neon
   - Resetar password do banco de dados
   - Atualizar variáveis de ambiente
```

### 4.2 - Criar Rotas API (2-3 horas)
**Endpoints a Implementar**:
- `POST /auth/register` - Registrar usuário
- `POST /auth/login` - Login
- `GET /families` - Listar famílias do usuário
- `POST /families` - Criar família
- `POST /families/:id/invite` - Convidar membro
- `GET /families/:id/members` - Listar membros
- `GET /accounts` - Listar contas
- `POST /accounts` - Criar conta
- `GET /bills` - Listar despesas
- `POST /bills` - Criar despesa

### 4.3 - Autenticação (1-2 horas)
**Implementar**:
- JWT com cookie/localStorage
- Session management
- Password hashing (bcrypt)
- Auth middleware

### 4.4 - Validação de Dados (1 hora)
**Usar**:
- Zod schemas já criados
- Middleware de validação
- Error handling padronizado

## Fase 5: Frontend React (Semana 2-3)

### 5.1 - Setup (1 hora)
- [ ] Create React App ou Vite
- [ ] Tailwind CSS
- [ ] React Router
- [ ] Supabase Client JS

### 5.2 - Páginas Principais (4-5 horas)
- [ ] Login/Registro
- [ ] Dashboard
- [ ] Gerenciar Famílias
- [ ] Contas (pessoal e familiar)
- [ ] Despesas e Transações

### 5.3 - Componentes (3-4 horas)
- [ ] Layout/Sidebar
- [ ] Forms
- [ ] Tables/Lists
- [ ] Modals
- [ ] Cards

### 5.4 - Integração API (2-3 horas)
- [ ] Conectar endpoints do backend
- [ ] State management (Redux/Context)
- [ ] Loading/Error states

## Fase 6: Testes (Semana 4)

### 6.1 - Testes Unitários
- [ ] Backend: Jest
- [ ] Frontend: Vitest

### 6.2 - Testes E2E
- [ ] Cypress ou Playwright

### 6.3 - Testes de RLS
- [ ] Verificar isolamento
- [ ] Testar permissões

## Fase 7: Deployment (Semana 4-5)

### 7.1 - Backend
- [ ] Deploy no Heroku ou Railway
- [ ] Variáveis de ambiente em produção

### 7.2 - Frontend
- [ ] Deploy no Vercel ou Netlify
- [ ] Build otimizado

### 7.3 - Banco de Dados
- [ ] Migrations
- [ ] Backups automáticos

## Verificação de Alinhamento com Realidade

### ✅ Documentação Alinhada com Supabase
- **8 tabelas**: Todas criadas no Supabase conforme documentação
- **RLS policies**: Habilitadas e funcionando
- **Schema**: Corresponde ao arquivo docs/SCHEMA.md
- **Constraints**: Implementados corretamente

### ⚠️ Documentação vs. Replit (Backend)
**Descobertas**:
1. Replit usa **Drizzle ORM** (não está documentado)
   - Alternativa mais moderna que raw SQL
   - Schema TypeScript com tipos seguros
   - Melhor para produção

2. Usar Enums extras (não estava no plano original):
   - `UserRole` - admin/member
   - `TransactionType` - income/expense
   - `BillStatus` - paid/pending/overdue
   - `RecurrenceType` - daily/weekly/monthly/etc

3. **Banco de Dados**: 
   - Estava usando Neon (PostgreSQL externo)
   - Mais eficiente: usar Supabase (já temos)

**Ações Necessárias**:
1. Atualizar `docs/SETUP.md` com instruções Drizzle ORM
2. Atualizar `docs/SCHEMA.md` com enums extras
3. Corrigir conexão banco de dados (Supabase vs Neon)

## Timeline Sugerida

```
Semana 1 (15-21 Nov):
  - Segunda: Resolver BD + Rotas API básicas
  - Terça-Quarta: Autenticação + Validação
  - Quinta-Sexta: Testes API

Semana 2-3 (22 Nov - 5 Dez):
  - Frontend setup + páginas principais
  - Integração com API
  - Componentes UI

Semana 4 (6-12 Dez):
  - Testes automatizados
  - Bug fixes
  - Polish UI/UX

Semana 5 (13-19 Dez):
  - Deploy staging
  - Deploy produção
  - Monitoramento
```

## Otimizações Recomendadas

### Para Próximas Sessões:
1. **Reutilize a documentação**: Não precisa reescrever
   - ARCHITECTURE.md já cobre tudo
   - SCHEMA.md é referência
   - SECURITY.md explica RLS

2. **Automação**: 
   - Usar `drizzle-kit` para migrations
   - Usar `npm run dev` para hot reload
   - CI/CD com GitHub Actions

3. **Banco de Dados**:
   - Migrar de Neon para Supabase (mais integrado)
   - Usar migrations programáticas
   - Backup automático

4. **TypeScript**:
   - Manter tipos sempre atualizado
   - Usar strict mode
   - Gerar tipos automaticamente do schema

## Conclusão

✅ **Base sólida**: Banco, RLS e documentação estão 100% funcionais
🎯 **Foco**: Agora é implementar a API e Frontend
⚡ **Eficiência**: Documentação bem feita acelera desenvolvimento
🔒 **Segurança**: RLS garantindo isolamento multi-tenant
