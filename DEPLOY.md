# Deploy-guide: Helpin (Next.js + Supabase)

Denne guiden tar deg gjennom å sette opp og deploye [mvpstack/helpin](https://github.com/mvpstack/helpin) — et selvhostet support-ticket-system bygget på Next.js + Supabase.

---

## 1. Skaff koden

Enklest: **fork repoet** på GitHub.

1. Gå til https://github.com/mvpstack/helpin
2. Trykk **Fork** øverst til høyre → velg kontoen din (`janheltne-afk`)
3. Du får nå `https://github.com/janheltne-afk/helpin`

Alternativt: klon lokalt og push til ditt eget repo.

```bash
git clone https://github.com/mvpstack/helpin.git
cd helpin
git remote set-url origin https://github.com/janheltne-afk/<ditt-repo>.git
git push -u origin main
```

---

## 2. Sett opp Supabase

### 2.1 Opprett prosjekt

1. Gå til https://supabase.com og logg inn / opprett konto.
2. Klikk **New project**, gi det et navn (f.eks. `helpin`), velg region nær brukerne dine, og sett et databasepassord (lagre dette).
3. Vent 1–2 minutter mens prosjektet provisjoneres.

### 2.2 Kjør SQL-schemaet

I Supabase-dashbordet:

1. Gå til **SQL Editor** (venstre sidemeny).
2. Åpne hver fil i `schema_query_backup/`-mappen fra repoet i denne rekkefølgen og kjør innholdet i editoren:
   - `users.sql`
   - `projectInfo.sql`
   - `tickets.sql`
   - `ticketMessages.sql`
   - `teams.sql`
3. Sjekk under **Table Editor** at tabellene `users`, `projectInfo`, `tickets`, `ticketMessages` og `teams` er opprettet.

### 2.3 Opprett storage bucket

Appen bruker en bucket kalt `assets` for fil-opplastinger:

1. Gå til **Storage** → **New bucket**.
2. Navn: `assets`. Sett **Public bucket** = på (eller styr via policies hvis du vil ha det privat).
3. Lag policies som tillater authenticated users å laste opp/lese.

### 2.4 Aktiver Realtime

Appen bruker Supabase Realtime for live chat:

1. Gå til **Database** → **Replication**.
2. Skru på replication for tabellen `ticketMessages`.

### 2.5 Hent API-nøkler

1. Gå til **Project Settings** → **API**.
2. Kopier:
   - **Project URL** → blir `NEXT_PUBLIC_SUPABASE_URL`
   - **anon public** key → blir `NEXT_PUBLIC_SUPABASE_ANON_KEY`

---

## 3. Test lokalt (valgfritt)

```bash
git clone https://github.com/janheltne-afk/helpin.git
cd helpin
cp .env.local.example .env.local
# fyll inn NEXT_PUBLIC_SUPABASE_URL og NEXT_PUBLIC_SUPABASE_ANON_KEY
npm install
npm run dev
```

Åpne http://localhost:3000.

### MaxMind GeoIP (valgfritt)

`config/ipLookup.js` og `pages/api/ipInfo.js` leser `ipData/asn.mmdb` og `ipData/city.mmdb`. De er ikke kritiske — appen funker uten dem hvis du ikke kaller `/api/ipInfo`. Hvis du vil ha IP-info:

1. Opprett gratis konto på https://www.maxmind.com/en/geolite2/signup
2. Last ned **GeoLite2-City** og **GeoLite2-ASN** (mmdb-format)
3. Legg filene i `ipData/`-mappen (de er ignored av `.gitignore`)

---

## 4. Deploy til Vercel

### 4.1 Importer prosjektet

1. Gå til https://vercel.com og logg inn (med GitHub for enkel kobling).
2. Klikk **Add New...** → **Project**.
3. Velg `janheltne-afk/helpin` (eller hva du har kalt repoet).
4. Vercel detekterer automatisk Next.js — la innstillingene være.

### 4.2 Sett miljøvariabler

Under **Environment Variables** legg til:

| Navn | Verdi |
| --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | (fra Supabase API-siden) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | (fra Supabase API-siden) |

### 4.3 Deploy

Trykk **Deploy**. Vercel bygger og gir deg en URL som `https://<prosjekt>.vercel.app`.

### 4.4 Konfigurer Supabase Auth

Tilbake i Supabase:

1. **Authentication** → **URL Configuration**.
2. Sett **Site URL** = `https://<prosjekt>.vercel.app`.
3. Legg til samme URL under **Redirect URLs**.

Hvis ikke vil signup/login redirecte feil.

---

## 5. Etter deploy

- Opprett en bruker via signup-flowen i appen.
- I Supabase → **Authentication** → **Users**: finn brukeren din.
- I **Table Editor** → tabellen `users`: endre `role` fra `customer` til noe annet (f.eks. `admin`) på din egen rad slik at du får tilgang til `/dashboard`.

---

## Feilsøking

- **"Invalid API key"**: dobbeltsjekk env-variablene i Vercel matcher Supabase-prosjektet.
- **Auth-redirect feiler**: sjekk Site URL i Supabase Auth-innstillingene.
- **Realtime-meldinger kommer ikke**: sjekk at replication er på for `ticketMessages`.
- **Build feiler i Vercel pga. ipData**: kommenter ut `pages/api/ipInfo.js` eller legg `.mmdb`-filene med under `ipData/` (ikke ideelt — store filer).

---

## Lenker

- Originalen: https://github.com/mvpstack/helpin
- Live demo: https://support.helpin.so
- Supabase docs: https://supabase.com/docs
- Vercel docs: https://vercel.com/docs
