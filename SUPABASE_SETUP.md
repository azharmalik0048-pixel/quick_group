# Supabase from the static React frontend

The quote form now sends directly from the React website to Supabase REST. No app server or Edge Function is needed for database inserts. The browser uses only the Supabase publishable key; Row Level Security allows quote inserts but denies public reads, updates, and deletes.

## 1. Enable insert-only access

The table was created earlier with public access revoked. Apply the second migration once in Supabase **SQL Editor**:

`supabase/migrations/20261111000200_allow_public_quote_inserts.sql`

This grants anonymous visitors insert access only to quote fields and validates the submitted values in the database. It does not grant table read access.

## 2. Supabase frontend values

The current Project URL and publishable key are in `.env.example` and are already the fallback values in `src/App.tsx`. If using another project, copy `.env.example` to `.env.local` and replace those two public values. Never put a service-role key, database password, or Apps Script secret in a `VITE_` variable.

Run the React website locally with `pnpm dev`; submit a test quote after applying the migration. It should appear in **Table Editor → quote_requests**. Build the static website with `pnpm run build` and upload `dist` to the frontend host.

## 3. Optional Google Sheet feed

Database inserts work without this step. To also copy new rows to the Sheet:

1. In `google-apps-script/Code.gs`, replace `REPLACE_WITH_A_LONG_RANDOM_SECRET` with a private random value, save, and update the existing web app deployment to a new version.
2. In Supabase, create a Database Webhook for `public.quote_requests`, event **INSERT**, method **POST**, targeting the Apps Script `/exec` URL with `?token=YOUR_SECRET` appended.
3. The Apps Script accepts Supabase's inserted `record` and appends it to the configured Sheet. Keep the token private.

The Supabase Edge Function files are no longer used by the React quote form.
