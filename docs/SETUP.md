# Instalação e Configuração - FamiliaFinanca

## Pré-requisitos

- Node.js 18+ ou superior
- npm ou yarn
- Conta Supabase (gratuita em supabase.com)
- Git

## Setup do Banco de Dados

### 1. Criar Projeto no Supabase

1. Acesse [supabase.com](https://supabase.com)
2. Crie uma nova conta ou faça login
3. Crie um novo projeto
4. Aguarde a inicialização

### 2. Executar Scripts SQL

1. Acesse o SQL Editor do seu projeto
2. Execute os scripts na seguinte ordem:
   - Criar todas as tabelas (schema)
   - Habilitar RLS em todas as tabelas
   - Criar as políticas de RLS

**Arquivo com scripts**: Ver `docs/SCHEMA.md` e `docs/SECURITY.md`

### 3. Copiar Credenciais

1. Acesse "Settings" → "API" no Supabase
2. Copie:
   - **Project URL**: `REACT_APP_SUPABASE_URL`
   - **Anon Key**: `REACT_APP_SUPABASE_ANON_KEY`

## Setup Local (Frontend)

### 1. Clonar Repositório

```bash
git clone https://github.com/2asconsultoriaetreinamento-lgtm/FamiliaFinanca.git
cd FamiliaFinanca
```

### 2. Instalar Dependências

```bash
npm install
# ou
yarn install
```

### 3. Criar .env.local

Crie arquivo `.env.local` na raiz do projeto:

```env
REACT_APP_SUPABASE_URL=https://seu-projeto.supabase.co
REACT_APP_SUPABASE_ANON_KEY=sua-chave-anonima
```

### 4. Executar em Desenvolvimento

```bash
npm start
# ou
yarn start
```

A aplicação será aberta em `http://localhost:3000`

### 5. Build para Produção

```bash
npm run build
# ou
yarn build
```

Os arquivos estáo prontos em `build/`

## Variáveis de Ambiente

### Desenvolvimento

```env
# Supabase
REACT_APP_SUPABASE_URL=https://seu-projeto.supabase.co
REACT_APP_SUPABASE_ANON_KEY=sua-chave-anonima

# Opcional - API Externa (futura)
REACT_APP_API_URL=http://localhost:3001
```

### Produção

- Configure as mesmas variáveis no seu host (Vercel, Netlify, etc.)
- Use o **Project URL e Anon Key** do Supabase em produção

## Testes

### Executar Testes Unitários

```bash
npm test
# ou
yarn test
```

### Executar Testes E2E

```bash
npm run test:e2e
# ou
yarn test:e2e
```

## Deploy

### Opção 1: Vercel (Recomendado)

1. Faça push do código para GitHub
2. Acesse [vercel.com](https://vercel.com)
3. Clique "New Project"
4. Selecione o repositório `FamiliaFinanca`
5. Adicione as variáveis de ambiente
6. Deploy automático ao fazer push

### Opção 2: Netlify

1. Faça push do código para GitHub
2. Acesse [netlify.com](https://netlify.com)
3. Clique "New site from Git"
4. Selecione repositório
5. Build command: `npm run build`
6. Publish directory: `build`
7. Configure variáveis de ambiente

### Opção 3: Heroku (Backend - Futuro)

```bash
heroku create familia-financa
git push heroku main
```

## Troubleshooting

### Erro: "REACT_APP_SUPABASE_URL is not set"

- Verifique se `.env.local` existe
- Confirme que as variáveis estão corretas
- Reinicie o servidor de desenvolvimento

### Erro: "Could not connect to database"

- Verifique a URL do Supabase
- Confirme que o projeto está ativo
- Cheque se a chave Anon é correta

### Erro: "RLS policy denied access"

- Verifique se o usuário está autenticado
- Confirme que o usuário é membro da família
- Veja `docs/SECURITY.md` para detalhes de RLS

### Erro: "Table does not exist"

- Execute os scripts SQL para criar as tabelas
- Verifique se está no projeto Supabase correto

## Estrutura de Pastas (Futuro)

```
FamiliaFinanca/
├── public/
├── src/
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   ├── services/
│   ├── utils/
│   ├── App.js
│   └── index.js
├── docs/
│   ├── ARCHITECTURE.md
│   ├── SCHEMA.md
│   ├── SECURITY.md
│   ├── SETUP.md
│   └── API.md
├── .env.local
├── package.json
├── README.md
└── .gitignore
```

## Próximos Passos

1. Implementar componentes React
2. Integrar Supabase Auth
3. Criar páginas principais
4. Implementar API de famílias
5. Testar RLS policies
6. Deploy em produção
