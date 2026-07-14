# Implementatiemonitor iWlz netwerkmodel

Webapplicatie waarmee softwareleveranciers de voortgang doorgeven van hun
implementatie richting het iWlz netwerkmodel. Gebouwd met Next.js, Supabase
(database + inlog) en Vercel (hosting).

Elke leverancier logt in met een eigen account en ziet/bewerkt alleen zijn
eigen voortgang. Admin-accounts zien een overzicht van alle leveranciers.
Dit wordt afgedwongen door Row Level Security in de database, niet alleen
door de app — dus ook veilig als iemand rechtstreeks op de database inlogt.

## Stap 1 — GitHub

1. Maak een account op https://github.com (gratis).
2. Maak een nieuw, **privé** repository aan, bijv. `implementatiemonitor-iwlz`.
3. Upload deze projectmap naar die repository (of vraag mij om dit via git
   voor je te doen zodra je de repo hebt aangemaakt).

## Stap 2 — Supabase (database + inloggen)

1. Maak een account op https://supabase.com en maak een nieuw project aan
   (kies een regio in de EU, bijv. Frankfurt, i.v.m. AVG).
2. Ga naar **SQL Editor** → **New query**, plak de inhoud van
   `supabase/migrations/0001_init.sql` en klik **Run**. Dit maakt alle
   tabellen, beveiligingsregels en een startlijst met implementatiefases aan.
3. Ga naar **Project settings → API** en noteer:
   - `Project URL` → wordt `NEXT_PUBLIC_SUPABASE_URL`
   - `anon public` key → wordt `NEXT_PUBLIC_SUPABASE_ANON_KEY`
4. Gebruikers aanmaken:
   - Ga naar **Authentication → Users → Add user** en maak een account aan
     per leverancier en per beheerder (met e-mail + wachtwoord). Elke nieuwe
     gebruiker krijgt automatisch een rij in de tabel `profiles` met rol
     `vendor`.
   - Ga naar **Table editor → vendors** en voeg een rij toe per
     softwareleverancier (naam, contactmail).
   - Ga naar **Table editor → profiles** en koppel elke gebruiker:
     - Voor een leverancier: zet `vendor_id` op het bijbehorende record uit
       `vendors`.
     - Voor een beheerder (bijv. jijzelf): zet `role` op `admin`.
5. Pas zo nodig de implementatiefases aan via **Table editor → phases**
   (volgorde, naam, omschrijving).

## Stap 3 — Vercel (hosting)

1. Maak een account op https://vercel.com, bij voorkeur inloggen met je
   GitHub-account.
2. Klik **Add New → Project** en kies de GitHub-repository die je in stap 1
   hebt aangemaakt.
3. Bij **Environment Variables** voeg je toe:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   (de waarden uit stap 2.3)
4. Klik **Deploy**. Na een paar minuten krijg je een live URL
   (bijv. `implementatiemonitor-iwlz.vercel.app`), later te vervangen door
   een eigen domein via **Project → Settings → Domains**.
5. Elke keer dat je naar de `main`-branch op GitHub pusht, deployt Vercel
   automatisch een nieuwe versie.

## Lokaal testen (optioneel, voor ontwikkelaars)

```bash
npm install
cp .env.example .env.local   # vul de Supabase-waarden in
npm run dev
```

App draait dan op http://localhost:3000.

## Projectstructuur

```
app/
  login/           Inlogpagina
  dashboard/        Overzicht voor admins (alle leveranciers x fases)
  vendor/            Voortgangsformulier voor een leverancier
  actions.ts         Server actions: inloggen, uitloggen, voortgang opslaan
lib/supabase/         Supabase client-helpers (browser, server, middleware)
middleware.ts          Beveiligt routes; niet-ingelogde gebruikers → /login
supabase/migrations/   SQL: tabellen, Row Level Security, startdata
```

## Volgende stappen / uitbreidingen

- Admin-UI om zelf leveranciers en gebruikers toe te voegen (nu via
  Supabase Table editor / Authentication).
- E-mailnotificaties bij statuswijziging (bijv. via Supabase Edge Functions
  of Resend).
- Bewijsstukken uploaden per fase (Supabase Storage).
- Exporteren naar Excel/PDF voor rapportage.
- Wachtwoord-resetflow en "wachtwoord vergeten" (Supabase Auth ondersteunt
  dit, moet nog aan de UI worden toegevoegd).
