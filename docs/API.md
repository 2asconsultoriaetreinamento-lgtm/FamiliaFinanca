# API Reference - FamiliaFinanca

## Visão Geral

Esta documentação descreve os endpoints da API do FamiliaFinanca. A API é baseada em Supabase com acesso via JavaScript Client.

**Status**: Em desenvolvimento
**Versão**: 0.1.0
**Base URL**: `https://seu-projeto.supabase.co`

## Autenticação

Todos os endpoints requerem autenticação via JWT token.

```javascript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.REACT_APP_SUPABASE_URL,
  process.env.REACT_APP_SUPABASE_ANON_KEY
)
```

## Endpoints

### 1. Users

#### Get Current User

```
GET /rest/v1/users?id=eq.<user-id>
```

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('users')
  .select('*')
  .eq('id', userId)
  .single()
```

**Response**:
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "full_name": "João Silva",
  "avatar_url": "https://...",
  "created_at": "2024-01-01T00:00:00Z"
}
```

#### Update User

```
PATCH /rest/v1/users?id=eq.<user-id>
```

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('users')
  .update({
    full_name: 'Novo Nome',
    avatar_url: 'https://...'
  })
  .eq('id', userId)
```

### 2. Families

#### Create Family

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('families')
  .insert([
    {
      owner_id: userId,
      name: 'Família Silva',
      description: 'Nossa família'
    }
  ])
  .select()
```

#### List User Families

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('families')
  .select('*')
  .eq('owner_id', userId)
```

#### Get Family with Members

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('families')
  .select(`
    *,
    family_members(
      user_id,
      role,
      users(*)
    )
  `)
  .eq('family_id', familyId)
  .single()
```

#### Update Family

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('families')
  .update({
    name: 'Novo Nome',
    description: 'Nova descrição'
  })
  .eq('family_id', familyId)
```

#### Delete Family

**JavaScript**:
```javascript
const { error } = await supabase
  .from('families')
  .delete()
  .eq('family_id', familyId)
```

### 3. Family Members

#### Add Member to Family

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('family_members')
  .insert([
    {
      family_id: familyId,
      user_id: newUserId,
      role: 'member'
    }
  ])
```

#### List Family Members

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('family_members')
  .select('*, users(*)')
  .eq('family_id', familyId)
```

#### Remove Member

**JavaScript**:
```javascript
const { error } = await supabase
  .from('family_members')
  .delete()
  .eq('family_id', familyId)
  .eq('user_id', userId)
```

### 4. Family Invites

#### Create Invite

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('family_invites')
  .insert([
    {
      family_id: familyId,
      invited_email: 'novo@example.com',
      inviter_id: currentUserId,
      status: 'pending'
    }
  ])
```

#### Accept Invite

**JavaScript**:
```javascript
// 1. Atualizar status do convite
const { error: inviteError } = await supabase
  .from('family_invites')
  .update({ status: 'accepted' })
  .eq('invite_id', inviteId)

// 2. Adicionar usuário aos membros
const { error: memberError } = await supabase
  .from('family_members')
  .insert([
    {
      family_id: familyId,
      user_id: currentUserId,
      role: 'member'
    }
  ])
```

### 5. Accounts

#### Create Account

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('accounts')
  .insert([
    {
      owner_id: userId,
      family_id: familyId, // null se pessoal
      name: 'Conta Corrente',
      is_family_account: false,
      balance: 0
    }
  ])
```

#### List User Accounts

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('accounts')
  .select('*')
  .eq('owner_id', userId)
```

#### List Family Accounts

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('accounts')
  .select('*')
  .eq('family_id', familyId)
  .eq('is_family_account', true)
```

### 6. Bills

#### Create Bill

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('bills')
  .insert([
    {
      account_id: accountId,
      creator_id: userId,
      family_id: familyId,
      description: 'Compras',
      amount: 150.00,
      is_family_bills: true
    }
  ])
```

#### List Bills

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('bills')
  .select('*')
  .eq('account_id', accountId)
  .order('created_at', { ascending: false })
```

### 7. Transactions

#### Create Transaction

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('transactions')
  .insert([
    {
      account_id: accountId,
      creator_id: userId,
      description: 'Depósito',
      amount: 1000.00,
      transaction_type: 'credit',
      is_family_transaction: true
    }
  ])
```

#### List Transactions

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('transactions')
  .select('*')
  .eq('account_id', accountId)
  .order('transaction_date', { ascending: false })
```

### 8. Activity Log

#### List Family Activity

**JavaScript**:
```javascript
const { data, error } = await supabase
  .from('family_activity_log')
  .select('*')
  .eq('family_id', familyId)
  .order('created_at', { ascending: false })
```

## Error Handling

### Common Errors

**42P01**: Table does not exist
- Solução: Verifique se as tabelas foram criadas

**42501**: RLS policy denied access
- Solução: Verifique se o usuário tem permissão

**23505**: Duplicate key value
- Solução: Valor já existe na tabela

**23503**: Foreign key violation
- Solução: Referência inválida

### Exemplo de Tratamento de Erro

```javascript
const { data, error } = await supabase
  .from('families')
  .select('*')

if (error) {
  console.error('Erro:', error.message)
  // Mostrar mensagem para o usuário
} else {
  console.log('Dados:', data)
}
```

## Rate Limiting

- Limite: 1000 requisições por minuto
- Header: `X-RateLimit-Remaining`

## Webhooks (Futuro)

Webhooks estarão disponíveis para eventos como:
- Novo membro adicionado
- Nova despesa criada
- Convite aceito

## Changelog

### v0.1.0 (Atual)
- Setup inicial com Supabase
- Tabelas do banco de dados
- RLS policies
- Documentação básica

### v0.2.0 (Planejado)
- Frontend React
- Autenticação
- Dashboard
- Gerenciamento de famílias
