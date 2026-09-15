# Contributing to Resonance

Start with README.md. Keep circle membership checks on every server operation.
Never include account credentials, user tokens, invitations, or private music
history in pull requests or logs. Submit focused changes with a description
and relevant validation. Use `npm run build` and `npx tsc --noEmit`.

The recommendation function lives in `lib/domain.ts`. Keep its explanations
faithful to actual signals. Do not present genre/artist overlap as measured
audio similarity. Any new metadata provider must respect its source terms.

For database changes, edit `db/schema.ts`, run `npm run db:generate`, inspect
the generated SQL, and append migrations after a release. Do not edit an
already deployed migration. Do not seed personal data in migrations.

Music adapters live in `lib/music.ts` and streaming exports in
`app/api/streaming/route.ts`. Provider-specific limits still apply to forks.

Contributions are licensed under the project's MIT license. Third-party
packages retain their own licenses.
