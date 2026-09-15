# Resonance

> **Updated redesign snapshot — September 15, 2026.** Start with
> [GITHUB-SETUP.md](GITHUB-SETUP.md) for the current feature status, repository
> upload instructions, and both database migrations. The new personal-web UI is
> included, but its automatic listening and remote-player backend is unfinished.
> Sections below describe the original circle-based implementation and should be
> read alongside that current status.

A shared music inbox, interactive taste map, and discovery playlists for groups of friends using Apple Music and Spotify. MIT licensed; see LICENSE. Third-party dependencies and vendored files retain their own licenses.

## What works

- Create multiple independent circles and switch between them.
- Invite people with cryptographically random, seven-day links. Only token hashes are stored. Owners can revoke all invitations or remove members.
- Persist shared song links, notes, genres, tags, and per-member reactions in Cloudflare D1.
- Explore song-to-artist, genre, and tag connections with a searchable, zoomable, pannable graph.
- Find catalog candidates near shared artists, ranked by each current member's positive/negative reactions and artist/genre overlap.
- Save discovery selections or mutual favorites as shared playlists; download their JSON track lists.
- Create streaming playlist snapshots after configuring provider credentials and authorizing an account. Missing or unavailable links are skipped and reported.

## What is not included

No credentials or private music data are bundled. Spotify catalog matching requires developer setup. Apple public catalog search works without it; mappings are user-confirmed. Search results are not guaranteed recording matches. Streaming editions do not continuously sync, and listening history is not imported. Discovery currently explores artists already present in a circle; it does not analyze audio or use Spotify Recommendations.

This source package is ready to publish in a repository but does not itself create a public GitHub repository.

## Local setup

Requires Node 22.13+ and npm. From the project root:

```sh
npm run install:ci
npm run build
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_organic_toad.sql
npm run dev
```

Open the URL printed by the server. Sign in uses the starter's loopback-only local identity, `Seedy`. Local storage lives in `.wrangler/state`, not browser localStorage. Apply each local migration only once. Subsequent releases append new migrations.

Copy `.env.example` to `.env` for local provider settings. For Cloudflare local preview, ensure these values are available as Worker bindings (Wrangler supports `.dev.vars`; do not commit it). Hosted Sites runtime secrets must be configured through the hosting control plane.

If npm's system shim is broken on Windows, run its actual JavaScript entrypoint with Node. If the sandbox prohibits npm's default cache, set `npm_config_cache` to a writable project-local directory. Keep caches and local state out of the source archive.

## Hosting and authentication

The included build targets Cloudflare Workers and D1. `.openai/hosting.json` declares the logical `DB` binding. Sites manages real resources and database migrations.

A new independent Sites deployment must register its own project ID. Do not reuse another instance's ID. The downloadable source package intentionally omits the original project's ID.

Production authentication is supplied by the Sites dispatcher through `oai-authenticated-user-*` headers. Every private API verifies identity and circle membership. The local mock strips visitor-supplied identity headers and only works on loopback; it is excluded from production builds.

**Do not deploy this Worker on an unprotected public origin that accepts caller-supplied identity headers.** On another host, replace `app/chatgpt-auth.ts` with a trusted session identity integration and supply the equivalent sign-in flow. Self-hosting outside Sites requires this integration, a D1 database, and migration deployment. The app is not a standalone public authentication server.

Site-level audience controls are separate from circle membership. To let new visitors join via link, the hosting audience must allow them to reach the app. Once signed in, only members can read a circle's music. Public app access never makes private circle records public.

## Optional provider setup

### Spotify

1. Create a developer app and set `SPOTIFY_CLIENT_ID` and `SPOTIFY_CLIENT_SECRET` as runtime secrets. The client ID is intentionally public; the client secret stays on the server.
2. Register the exact site origin followed by `/` as a redirect URI. For local development use a Spotify-supported loopback redirect and consistent host; check Spotify's current redirect policy.
3. Add Spotify accounts to the developer app allowlist. Spotify currently limits new development-mode apps to five users and requires Premium for the app owner. Open sourcing this app does not bypass those restrictions.
4. Catalog resolution uses client credentials. Streaming export uses Authorization Code with PKCE and `playlist-modify-private`, with state/verifier stored only for the short authorization flow. Account tokens are used for that export and are not saved in D1 or browser storage.
5. Streaming export only uses confirmed Spotify track links. Apple-only discovery playlists must have Spotify recordings confirmed before they can be exported there.

### Apple Music

1. Create a MusicKit key and sign a developer token according to Apple documentation.
2. Set `APPLE_MUSIC_DEVELOPER_TOKEN` and renew it before expiration. It is intentionally exposed to signed-in browsers for MusicKit, unlike the private signing key, which this app never needs.
3. Each listener authorizes MusicKit when exporting. The server checks the listener's storefront and available catalog recordings before creating a library playlist.
4. A MusicKit subscription and developer configuration are required to test real playlist writes. This build has no credentials preconfigured.

Provider API calls use fixed approved hosts, timeouts, and errors surfaced to the user. The app never downloads full audio. iTunes metadata is used to link to the catalog, not as an audio similarity dataset.

## Architecture

- `app/resonance.tsx`: group interface and interactive graph.
- `app/api/circles/route.ts`: persistent group operations and discovery.
- `lib/domain.ts`: portable types and explainable ranking.
- `lib/music.ts`: catalog lookup and recording candidates.
- `app/api/streaming/route.ts`, `lib/streaming-client.ts`: optional provider authorization and playlist export.
- `db/schema.ts`, `drizzle/`: schema and versioned migrations.
- `app/chatgpt-auth.ts`: replaceable trusted identity boundary.

The graph shows metadata connections. Song colors are decorative and do not represent members or inferred similarity scores. A song sent by someone is not automatically counted as their positive reaction. Unheard is neutral; removed members' reactions no longer influence recommendations.

## Validation

Run `npx tsc --noEmit` and `npm run build`. See CONTRIBUTING.md. The initial build was exercised through local API integration tests covering authentication, origin checks, circle membership, invitation revocation, persisted songs/reactions/playlists, and input validation. Real provider authorization and playlist creation require credentials and have not been end-to-end tested on live accounts. Browser visual QA and WebMCP execution were not performed.

## Reference documentation

- https://developer.spotify.com/documentation/web-api/concepts/quota-modes
- https://developer.spotify.com/documentation/web-api/tutorials/code-pkce-flow
- https://developer.apple.com/musickit/
- https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/iTuneSearchAPI/index.html
