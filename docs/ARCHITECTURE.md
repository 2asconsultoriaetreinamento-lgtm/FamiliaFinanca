# Arquitetura do Sistema - FamiliaFinanca

## Visão Geral

O FamiliaFinanca é um sistema de gestão financeira familiar com arquitetura multi-tenant, projetado para permitir que famílias gerenciem suas despesas de forma centralizada e colaborativa, enquanto mantém controle granular sobre dados pessoais e familiares.

## Arquitetura Multi-Tenant

A aplicação implementa um modelo multi-tenant baseado em **famílias**, onde:

- Cada **família** representa um tenant isolado
- Usuários podem pertencer a múltiplas famílias
- Dados são isolados por família através de Row Level Security (RLS)
- O proprietário da família controla quem pode acessar os dados

## Componentes Principais

### 1. Autenticação e Autorização

- **Autenticação**: Supabase Auth (JWT tokens)
- **Autorização**: Row Level Security (RLS) + Políticas de Acesso
- **Controle de Acesso**: Baseado em:
  - Propriedade do usuário (usuário logado)
  - Associação à família
  - Tipo de registro (pessoal vs. familiar)

### 2. Gerenciamento de Famílias

**Fluxo de Criação:**

1. Usuário cria uma nova família
2. Usuário é definido como proprietário (owner_id)
3. Usuário pode convidar membros via convites
4. Membros aceitam convites e são adicionados à família

**Controle de Acesso:**

- Apenas o proprietário pode gerenciar membros
- Apenas o proprietário pode deletar a família
- Membros da família podem visualizar dados familiares

### 3. Tipo de Registros

O sistema diferencia dois tipos de registros:

#### Registros Pessoais (is_family_* = false)

- Propriedade exclusiva do usuário
- Visibilidade: apenas o criador pode visualizar
- Uso: contas, transações e despesas individuais
- RLS: Comparação de usuário logado (auth.uid()) com criador

#### Registros Familiares (is_family_* = true)

- Propriedade compartilhada pela família
- Visibilidade: todos os membros da família
- Uso: contas compartilhadas, despesas coletivas
- RLS: Verificação de membros via tabela family_members

### 4. Tabelas Principais

**users**
- Dados do usuário
- id: UUID (chave primária, vem do Supabase Auth)
- email, full_name, avatar_url

**families**
- Definição de famílias
- family_id: UUID (chave primária)
- owner_id: UUID (proprietário, referencia users.id)
- name, description
- created_at, updated_at

**family_members**
- Relacionamento M2M entre usuários e famílias
- family_id + user_id: chave composta única
- role: papel do membro na família

**family_invites**
- Convites para membros
- Gerenciamento de status: pending, accepted, rejected
- Link para nova adição de membros

**accounts**
- Contas bancárias/financeiras
- is_family_account: diferencia conta pessoal vs. familiar
- Referência a family_id quando familiar

**bills e transactions**
- Movimentações financeiras
- is_family_*: tipo de registro
- Referência a family_id quando familiar

**family_activity_log**
- Log de atividades da família
- Auditoria de mudanças
- Visibilidade apenas para membros

## Fluxo de Segurança

### 1. Isolamento de Dados

```
User A (Family 1)  ──┐
User B (Family 1)  ──┼─→ Family 1 Data (Isolated)
User C (Family 2)  ──┤
User D (Family 2)  ──┴─→ Family 2 Data (Isolated)
```

### 2. Row Level Security (RLS)

Todas as tabelas têm RLS habilitado com políticas que:

- Verificam se o usuário logado (auth.uid()) tem permissão
- Validam associação à família quando necessário
- Diferenciam entre registros pessoais e familiares

**Exemplo de Política (Accounts):**

```sql
-- Visualizar conta pessoal
SELECT (auth.uid())::text = owner_id::text 
  AND is_family_account = false

-- Visualizar conta familiar
SELECT EXISTS (
  SELECT 1 FROM family_members fm
  WHERE fm.family_id = accounts.family_id
  AND fm.user_id = (auth.uid())::text
)
AND accounts.is_family_account = true
```

## Fluxos de Caso de Uso

### 1. Usuário Cria Familia e Convida Membros

1. User A cria "Família Silva"
   - owner_id = User A
   - User A é adicionado a family_members

2. User A convida User B
   - Cria registro em family_invites (status: pending)
   - Email enviado para User B

3. User B aceita convite
   - family_invites.status = accepted
   - User B adicionado a family_members
   - User B pode visualizar dados da "Família Silva"

### 2. Gerenciar Despesas Familiares

1. User A cria conta compartilhada
   - Cria accounts com is_family_account = true
   - Referencia Família Silva via family_id

2. User A registra despesa familiar
   - Cria bills com is_family_bills = true
   - Referencia account e family_id

3. User B acessa despesa
   - RLS valida: User B é membro de Família Silva
   - User B visualiza despesa

### 3. Contas Pessoais Isoladas

1. User A cria conta pessoal
   - Cria accounts com is_family_account = false

2. User B tenta visualizar
   - RLS bloqueia: não é dono e é pessoal
   - User B não vê a conta

## Escalabilidade

- **Isolamento por família**: Cada query é filtrado por family_id
- **Índices**: family_id indexado em todas as tabelas
- **Particionamento futuro**: Possível particionar tabelas por family_id
- **Cache**: Dados de membros podem ser cacheados

## Segurança

- ✅ Autenticação: Supabase JWT
- ✅ Isolamento: RLS + Políticas
- ✅ Encriptação: Supabase padrão
- ✅ Auditoria: family_activity_log
- ✅ Validação: Constraints no banco de dados

## Próximos Passos

1. Implementar frontend (React/Vue)
2. Criar API endpoints
3. Implementar convites por email
4. Adicionar autenticação social
5. Desenvolver dashboard de analytics
