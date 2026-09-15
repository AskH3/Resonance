# Resonance — updated source (September 15, 2026)

## Upload to your repository

Unzip this archive and upload the **contents** of the `resonance` folder to the
root of your GitHub repository. `package.json` should be at the repository root.
Include dotfiles such as `.gitignore`, `.env.example`, and `.openai/hosting.json`.
Do not upload your real `.env`, account credentials, `node_modules`, or local database.

This archive contains the updated app source, not just an HTML file. A GitHub
repository stores code. GitHub Pages only hosts static files and cannot run this
app's authentication, database, or server API routes. For the full app, connect
the repository to a compatible Cloudflare Workers/D1 deployment with a trusted
authentication integration. See the authentication boundary in README.md.

## Run locally

Use Node.js 22.13 or later. In the project folder:

```sh
npm run install:ci
npm run build
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_organic_toad.sql
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0001_mysterious_lake.sql
npm run dev
```

Apply each migration only once. On an existing local database that already has
the first migration, apply only `0001_mysterious_lake.sql`. Open the URL printed
by the development server. Local sign-in is a loopback-only development identity;
do not expose it publicly as a production login service.

## Included in this update

- Black-and-lime redesign with the centered Resonance logo.
- My Web home screen; Discover and People navigation.
- Rotating, draggable constellation with perspective depth and real catalog cover art.
- Touch rotation, pinch zoom, mobile layouts, and a persistent player interface.
- Personal web storage, scoped friend/circle webs, and personal/shared playlist records.
- YouTube Music Google Takeout JSON parsing and import path.
- Existing circle management under `/circles`.
- Source, database migrations, MIT license, and contribution instructions.

## Work still in progress — do not mistake the UI for a live integration

The new `/api/listening` and `/api/listening/connect` backend routes are **not
implemented in this snapshot**. Automatic listening synchronization, live player
controls, persistent account connections, presence refresh, and the new web's
Spotify playlist-send action are therefore not operational. Supplying credentials
alone does not finish these new routes. `finishConnection()` is a placeholder.

The older `/api/streaming` playlist export flow exists separately under circle
management and requires provider credentials. It has not been tested with live
accounts. YouTube Music link resolution is not yet implemented in the catalog
resolver; use the Takeout import path for this snapshot. Import accepts only
entries identifiable as YouTube Music and will skip generic YouTube history.

Example album covers are labeled catalog previews; they are not your listening
history. Live sharing and personal-data permissions need further end-to-end
testing before public use. This is an updated development source handoff, not a
finished production service.

## Checks

`npx tsc --noEmit` checks the source types. `npm test` checks the existing ranking
logic. A successful build does not verify missing provider endpoints, real music
account authorization, mobile browser behavior, or production authentication.

GitHub Pages reference:
https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages
