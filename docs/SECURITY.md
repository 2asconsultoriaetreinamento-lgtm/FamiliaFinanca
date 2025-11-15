# Segurança e RLS - FamiliaFinanca

## Visão Geral de Segurança

O FamiliaFinanca implementa múltiplas camadas de segurança:

1. **Autenticação**: Supabase Auth com JWT
2. **Autorização**: Row Level Security (RLS) no banco de dados
3. **Isolamento**: Múltiplas famílias isoladas
4. **Auditoria**: Activity logs de todas as ações

## Row Level Security (RLS)

### Visão Geral

RLS é implementado em todas as 8 tabelas para garantir que:
- Usuários só vejam dados que têm permissão
- Dados pessoais são isolados
- Dados familiares são compartilhados apenas entre membros
- O proprietário da família tem controle total

### Políticas por Tabela

#### 1. Tabela: users

**Política: users_view_own**
- Tipo: SELECT
- Condição: `(auth.uid())::text = id`
- Efeito: Usuários veem apenas seus próprios dados

**Política: users_update_own**
- Tipo: UPDATE
- Condição: `(auth.uid())::text = id`
- Efeito: Usuários atualizam apenas suas contas

**Política: users_insert_own**
- Tipo: INSERT
- Condição: `(auth.uid())::text = id`
- Efeito: Usuários inserem apenas suas contas

#### 2. Tabela: families

**Política: families_owner_view**
- Tipo: SELECT
- Condição: `(auth.uid())::text = owner_id`
- Efeito: Proprietários veem suas famílias

**Política: families_members_view**
- Tipo: SELECT
- Condição: Membro da família via family_members
- Efeito: Membros veem a família

```sql
EXISTS (
  SELECT 1 FROM family_members fm
  WHERE fm.family_id = families.family_id
  AND fm.user_id = (auth.uid())::text
)
```

#### 3. Tabela: family_members

**Políticas**: 4 políticas para SELECT, INSERT, UPDATE, DELETE

Todas verificam se o usuário é membro da família:

```sql
EXISTS (
  SELECT 1 FROM family_members fm
  WHERE fm.family_id = family_members.family_id
  AND fm.user_id = (auth.uid())::text
)
```

#### 4. Tabela: accounts

**Política: accounts_view_personal**
- Tipo: SELECT
- Condição: `(auth.uid())::text = owner_id AND is_family_account = false`
- Efeito: Contas pessoais visíveis apenas para o dono

**Política: accounts_view_family**
- Tipo: SELECT
- Condição: Membro da família E `is_family_account = true`
- Efeito: Contas familiares visíveis para membros

**Políticas**: INSERT, UPDATE, DELETE similares com mesmas condições

#### 5. Tabela: bills

Seguem mesmo padrão da tabela accounts:
- Despesas pessoais: vistas apenas pelo criador
- Despesas familiares: vistas por membros da família

#### 6. Tabela: transactions

Idem bills - padrão pessoal vs. familiar

#### 7. Tabela: family_invites

**Política: family_invites_view**
- Tipo: SELECT
- Condição: Proprietário da família OU convidado
- Efeito: Apenas interessados veem convites

#### 8. Tabela: family_activity_log

**Política: activity_log_view**
- Tipo: SELECT
- Condição: Membro da família
- Efeito: Membros veem atividades da família

## Regras de Acesso

### Registros Pessoais

Campo: `is_family_* = FALSE` ou `NULL` em family_id

**Permissões:**
- Criar: Apenas o próprio usuário
- Visualizar: Apenas o criador/dono
- Editar: Apenas o criador/dono
- Deletar: Apenas o criador/dono

**Exemplo: Conta Pessoal**
```
User A cria conta pessoal
  - is_family_account = false
  - owner_id = User A
  - family_id = NULL

User A vê? SIM (RLS permite)
User B vê? NÃO (RLS bloqueia)
User C vê? NÃO (RLS bloqueia)
```

### Registros Familiares

Campo: `is_family_* = TRUE` + `family_id = <uuid>`

**Permissões:**
- Criar: Membro da família
- Visualizar: Todos os membros
- Editar: Depende de role/ownership
- Deletar: Depende de role/ownership

**Exemplo: Conta Familiar**
```
User A cria conta familiar na Família Silva
  - is_family_account = true
  - family_id = <uuid-familia-silva>
  - owner_id = User A

User A (membro) vê? SIM (RLS permite)
User B (membro) vê? SIM (RLS permite)
User C (não membro) vê? NÃO (RLS bloqueia)
```

## Fluxo de Isolamento

### 1. Autenticação
```
Client
  |
  v
 Supabase Auth (JWT)
  |
  v
Database (auth.uid() disponível)
```

### 2. Validação RLS
```
Query (SELECT * FROM accounts)
  |
  v
RLS Policy Check
  |
  +-- is_family_account = false?
  |   v
  |   (auth.uid())::text = owner_id? -> SIM/NÃO
  |
  +-- is_family_account = true?
      v
      Member check via family_members? -> SIM/NÃO
```

## Type Casting

### Problema
`auth.uid()` retorna UUID, mas colunas podem ser TEXT ou INTEGER

### Solução
Converter ambos os lados para TEXT:

```sql
-- ERRADO: operator does not exist: uuid = text
auth.uid() = user_id

-- CORRETO: cast para text
(auth.uid())::text = user_id
(auth.uid())::text = (id::text)
```

## Auditoria

### family_activity_log

Todas as ações importantes são registradas:

```sql
INSERT INTO family_activity_log (
  family_id, user_id, action, description
) VALUES (
  '<family-id>',
  '<user-id>',
  'created_bill',
  'Criou despesa: Compras (R$ 150.00)'
);
```

### Eventos Auditados
- Criação de família
- Adição de membro
- Remoção de membro
- Criação de despesa
- Atualização de conta
- Exclusão de registros

## Boas Práticas de Segurança

### Frontend
1. **Não confiar em JWT**: Sempre validar no backend
2. **HTTPS obrigatório**: Encriptar dados em trânsito
3. **Validar entrada**: Não executar SQL direto
4. **Gerenciar tokens**: Limpar ao logout

### Backend
1. **RLS habilitado**: Sempre usar em produção
2. **Validar contexto**: auth.uid() deve estar presente
3. **Auditoria**: Registrar ações sensibilizadas
4. **Rate limiting**: Proteger contra abuso

### Banco de Dados
1. **Backups regulares**: Preparar para desastres
2. **Monitoring**: Alertar sobre comportamentos anormais
3. **Encriptação**: Em repouso e em trânsito
4. **Constraints**: Validar integridade dos dados

## Próximos Passos de Segurança

- [ ] Implementar 2FA (two-factor authentication)
- [ ] Adicionar rate limiting
- [ ] Implementar CAPTCHA
- [ ] Monitoramento de atividades suspeitas
- [ ] Encryptação de dados sensibilizados
- [ ] GDPR compliance
- [ ] Política de privacidade
