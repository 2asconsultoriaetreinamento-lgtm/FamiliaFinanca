# FamiliaFinanca

> Sistema de gestao financeira familiar multi-tenant com suporte a contas pessoais e compartilhadas

## Sobre o Projeto

FamiliaFinanca eh uma aplicacao web moderna para gerenciar financas familiares com seguranca e controle granular de acesso.

## Principais Caracteristicas

- Contas Compartilhadas: Compartilhe transacoes e contas a pagar com membros da familia
- Isolamento de Dados: Registros pessoais sao visiveis apenas ao criador
- Seguranca em Camadas: Row Level Security (RLS) no banco de dados
- Auditoria: Log de todas as atividades familiares
- Multi-Tenant: Suporta multiplas familias independentes

## Documentacao

- [Arquitetura do Sistema](./docs/ARCHITECTURE.md)
- [Schema do Banco de Dados](./docs/SCHEMA.md)
- [Seguranca e RLS](./docs/SECURITY.md)
- [Guia de Instalacao](./docs/SETUP.md)

## Stack Tecnologico

- Frontend: React/Next.js com TypeScript
- Backend: Node.js com Replit
- Banco de Dados: Supabase PostgreSQL
- ORM: Drizzle ORM

## Licenca

MIT - Veja o arquivo LICENSE para mais detalhes.
