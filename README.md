# Ticketsystem

A self-hosted support ticketing app, based on [mvpstack/helpin](https://github.com/mvpstack/helpin) (Next.js + Supabase).

![Preview](https://raw.githubusercontent.com/mvpstack/helpin/main/dashboard-preview.png)

## Setup

Se [DEPLOY.md](DEPLOY.md) for full deploy-instruksjoner (Supabase + Vercel).

Kort versjon:

1. Opprett et Supabase-prosjekt og kjør alle SQL-filene i `schema_query_backup/` i SQL-editoren.
2. Kopier `.env.local.example` til `.env.local` og fyll inn `NEXT_PUBLIC_SUPABASE_URL` og `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
3. `npm install && npm run dev`
4. Deploy til Vercel med de samme env-variablene satt.
