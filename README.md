# Loovi · Ressarcimento — Painel (Vercel + Supabase)

Site estático (sem build) com visual do Design System TRIA (tema escuro + neon `#B5FF00`).
O **frontend** fica na Vercel; os **34.254 registros** ficam no **Supabase** (Postgres),
consultados em tempo real com busca e paginação no servidor.

## Telas
- **Painel** — KPIs, gráfico por analista e tabela consolidada (lê `data/summary.json`, um snapshot leve).
- **Registros** — explorador por analista/período, com busca e paginação **server-side** (Supabase),
  e modal de detalhe que carrega o documento completo (todas as 60 colunas) sob demanda.

## Estrutura
```
loovi-web/
├── index.html          ← app inteira (HTML + CSS + JS, sem dependências)
├── vercel.json         ← config de deploy
├── data/
│   ├── summary.json    ← KPIs/resumo do Painel (snapshot)
│   └── index.json      ← lista de analistas/períodos (dropdown dos Registros)
└── README.md
```

## Configuração (já preenchida no index.html)
No topo do `<script>` em `index.html`:
```js
const SB_URL = 'https://uytizgmncjbbbzkflimb.supabase.co';
const SB_KEY = 'sb_publishable_...';   // chave PUBLICÁVEL (pode ficar exposta)
```
> A `service/secret` key **nunca** está no frontend — foi usada só para carregar os dados.
> A tabela `loovi_ressarcimento` tem **RLS com política só-leitura** (anon/publishable não escreve).

## Subir na Vercel
**Opção A — web:** vercel.com → Add New → Project → Deploy → arraste a pasta `loovi-web`
→ framework **Other** → Deploy.
**Opção B — CLI:**
```bash
npm i -g vercel
cd loovi-web
vercel --prod
```
Sem etapa de build (Build Command vazio, Output Directory = `.`).

## Rodar localmente
```bash
cd loovi-web
python -m http.server 8000   # abra http://localhost:8000
```

## Atualizar os dados (recarga)
Os dados vieram de `2025 - Loovi Ressarcimento - ORGANIZADO.xlsx` via script de carga
(`loovi_load.py`, insere em lotes na API do Supabase). Para recarregar:
1. `truncate table public.loovi_ressarcimento;` (SQL Editor)
2. rodar o script de carga novamente
3. (opcional) regerar `data/summary.json` e `data/index.json` se as contagens mudarem.

## Segurança — pendências recomendadas
- **Gire a `service/secret` key** do projeto Loovi (foi compartilhada no chat durante a carga).
- O projeto **Auton - DB** (médico) tem **RLS desabilitado em 56 tabelas** — qualquer um com a
  anon key daquele projeto lê dados sensíveis. Tratar separadamente (não faz parte deste painel).
