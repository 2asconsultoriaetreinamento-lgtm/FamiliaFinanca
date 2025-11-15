# Schema do Banco de Dados - FamiliaFinanca

## Visão Geral

O banco de dados FamiliaFinanca utiliza PostgreSQL através do Supabase com 8 tabelas principais que implementam uma arquitetura multi-tenant baseada em famílias.

## Tabelas

### 1. users

Tabela de usuários sincronizada com Supabase Auth.

```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `id` (TEXT): UUID do usuário (vem do Supabase Auth)
- `email` (TEXT): Email único do usuário
- `full_name` (TEXT): Nome completo
- `avatar_url` (TEXT): URL da foto de perfil
- `created_at` (TIMESTAMP): Data de criação
- `updated_at` (TIMESTAMP): Data de última atualização

### 2. families

Tabela de famílias (tenants).

```sql
CREATE TABLE families (
  family_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `family_id` (UUID): Identificador único da família
- `owner_id` (TEXT): ID do proprietário (referencia users.id)
- `name` (TEXT): Nome da família
- `description` (TEXT): Descrição/notário
- `created_at` (TIMESTAMP): Data de criação
- `updated_at` (TIMESTAMP): Data de última atualização

### 3. family_members

Tabela de associação M2M entre usuários e famílias.

```sql
CREATE TABLE family_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  family_id UUID NOT NULL REFERENCES families(family_id) ON DELETE CASCADE,
  user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role TEXT DEFAULT 'member',
  joined_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(family_id, user_id)
);
```

**Colunas:**
- `id` (UUID): Identificador único do registro
- `family_id` (UUID): ID da família
- `user_id` (TEXT): ID do usuário
- `role` (TEXT): Papel do membro (member, admin, etc.)
- `joined_at` (TIMESTAMP): Data de aderéncia
- **Constraint**: UNIQUE(family_id, user_id) - cada usuário pode pertencer uma única vez a cada família

### 4. family_invites

Tabela de convites para membros.

```sql
CREATE TABLE family_invites (
  invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  family_id UUID NOT NULL REFERENCES families(family_id) ON DELETE CASCADE,
  invited_email TEXT NOT NULL,
  inviter_id TEXT NOT NULL REFERENCES users(id),
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `invite_id` (UUID): Identificador único do convite
- `family_id` (UUID): ID da família convidadora
- `invited_email` (TEXT): Email do convidado
- `inviter_id` (TEXT): ID do usuário que enviou o convite
- `status` (TEXT): Status do convite (pending, accepted, rejected)
- `created_at` (TIMESTAMP): Data de criação
- `updated_at` (TIMESTAMP): Data de atualização

### 5. accounts

Tabela de contas financeiras.

```sql
CREATE TABLE accounts (
  account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  family_id UUID REFERENCES families(family_id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  balance DECIMAL(15,2) DEFAULT 0,
  is_family_account BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `account_id` (UUID): Identificador único da conta
- `owner_id` (TEXT): ID do criador da conta
- `family_id` (UUID): ID da família (NULL se pessoal)
- `name` (TEXT): Nome da conta
- `description` (TEXT): Descrição
- `balance` (DECIMAL): Saldo atual
- `is_family_account` (BOOLEAN): Indica se é conta familiar
- `created_at` (TIMESTAMP): Data de criação
- `updated_at` (TIMESTAMP): Data de última atualização

**RLS Rules:**
- Se `is_family_account = FALSE`: apenas `owner_id` pode visualizar/editar
- Se `is_family_account = TRUE`: membros de `family_id` podem visualizar

### 6. bills

Tabela de despesas/contas.

```sql
CREATE TABLE bills (
  bill_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
  creator_id TEXT NOT NULL REFERENCES users(id),
  family_id UUID REFERENCES families(family_id) ON DELETE CASCADE,
  description TEXT NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  due_date DATE,
  is_family_bills BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `bill_id` (UUID): Identificador único
- `account_id` (UUID): ID da conta
- `creator_id` (TEXT): ID do criador
- `family_id` (UUID): ID da família (NULL se pessoal)
- `description` (TEXT): Descrição da despesa
- `amount` (DECIMAL): Valor
- `due_date` (DATE): Data de vencimento
- `is_family_bills` (BOOLEAN): Indica se é despesa familiar
- `created_at` (TIMESTAMP): Data de criação
- `updated_at` (TIMESTAMP): Data de última atualização

### 7. transactions

Tabela de transações/movimentações.

```sql
CREATE TABLE transactions (
  transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
  creator_id TEXT NOT NULL REFERENCES users(id),
  family_id UUID REFERENCES families(family_id) ON DELETE CASCADE,
  description TEXT NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  transaction_type TEXT NOT NULL,
  transaction_date DATE DEFAULT CURRENT_DATE,
  is_family_transaction BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `transaction_id` (UUID): Identificador único
- `account_id` (UUID): ID da conta
- `creator_id` (TEXT): ID do criador
- `family_id` (UUID): ID da família (NULL se pessoal)
- `description` (TEXT): Descrição
- `amount` (DECIMAL): Valor
- `transaction_type` (TEXT): Tipo (debit, credit, transfer)
- `transaction_date` (DATE): Data da transação
- `is_family_transaction` (BOOLEAN): Indica se é transação familiar
- `created_at` (TIMESTAMP): Data de criação
- `updated_at` (TIMESTAMP): Data de última atualização

### 8. family_activity_log

Tabela de log de atividades da família.

```sql
CREATE TABLE family_activity_log (
  log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  family_id UUID NOT NULL REFERENCES families(family_id) ON DELETE CASCADE,
  user_id TEXT NOT NULL REFERENCES users(id),
  action TEXT NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Colunas:**
- `log_id` (UUID): Identificador único
- `family_id` (UUID): ID da família
- `user_id` (TEXT): ID do usuário que realizou a ação
- `action` (TEXT): Tipo de ação (created_bill, added_member, etc.)
- `description` (TEXT): Descrição detalhada
- `created_at` (TIMESTAMP): Data da ação

## Índices

```sql
CREATE INDEX idx_families_owner ON families(owner_id);
CREATE INDEX idx_family_members_family ON family_members(family_id);
CREATE INDEX idx_family_members_user ON family_members(user_id);
CREATE INDEX idx_family_invites_family ON family_invites(family_id);
CREATE INDEX idx_accounts_owner ON accounts(owner_id);
CREATE INDEX idx_accounts_family ON accounts(family_id);
CREATE INDEX idx_bills_account ON bills(account_id);
CREATE INDEX idx_bills_family ON bills(family_id);
CREATE INDEX idx_transactions_account ON transactions(account_id);
CREATE INDEX idx_transactions_family ON transactions(family_id);
CREATE INDEX idx_activity_log_family ON family_activity_log(family_id);
```

## Relações

```
users
  |
  ├── families (owner_id)
  |
  ├── family_members (user_id)
  |
  ├── accounts (owner_id)
  |
  ├── bills (creator_id)
  |
  ├── transactions (creator_id)
  |
  └── family_activity_log (user_id)

families
  |
  ├── family_members (family_id)
  |
  ├── family_invites (family_id)
  |
  ├── accounts (family_id)
  |
  ├── bills (family_id)
  |
  ├── transactions (family_id)
  |
  └── family_activity_log (family_id)
```

## Constraints

### Foreign Keys
- Todos com ON DELETE CASCADE para garantir limpeza de dados orfanatos

### Unique Constraints
- `users.email`: Garante emails únicos
- `family_members(family_id, user_id)`: Garante que um usuário só pode estar em uma família uma vez

### Default Values
- `balance = 0` em accounts
- `is_family_account = FALSE` em accounts
- `is_family_bills = FALSE` em bills
- `is_family_transaction = FALSE` em transactions

## Normalização

O schema segue 3NF (Terceira Forma Normal):
- Sem dependências transitivas
- Todas as chaves estrangeiras referenciam chaves primárias
- Sem átomos repetidos (exceto tipos de dados)
