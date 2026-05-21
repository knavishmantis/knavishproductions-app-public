# knavishproductions-app

Production pipeline app for the [KnavishMantis](https://youtube.com/@knavishmantis) YouTube channel. Live at [knavishproductions.com](https://knavishproductions.com). Next.js + Postgres on Cloud Run + Firebase Hosting.

> Source is private. This README is the public-facing architecture writeup.

Closes the loop on the channel's content pipeline: takes validated briefs from [prospecting-engine](https://github.com/knavishmantis/prospecting-engine-public), runs them through scripting, asset handoff to paid editors, and payment tracking. Replaces a scattered toolchain (one tool per stage) with a single app where shorts move through the workflow end-to-end.

```mermaid
flowchart TB
  User[Browser · knavishproductions.com]
  Auth[Google OAuth]
  GHA[GitHub Actions]
  PE[prospecting-engine · Cloud Run]

  User --> FB[Firebase Hosting · CDN]
  FB --> CR[Cloud Run · Next.js app]
  CR --> SQL[(Cloud SQL · Postgres)]
  CR --> Assets[(GCS · assets)]
  CR --> Briefs[(GCS · briefs)]
  CR --> SM[Secret Manager]
  CR --> Auth
  GHA -->|deploy| CR
  PE -->|writes| SQL
  PE -->|writes| Briefs
```

Five modules behind a single Next.js App Router app:

1. **Brief inbox** — consumer side of the producer/consumer contract with `prospecting-engine`. Reads briefs from a Postgres table + GCS prefix the engine writes to; the engine never reads anything the app writes.
2. **Script editor** — Tiptap-based rich editor for turning a brief into a filmable script. Per-short revisions, comments.
3. **Asset manager** — uploaded clips, b-roll, source files keyed to a short.
4. **Editor handoff** — packages a script + assets into a workable bundle for a paid editor; tracks status from sent → revised → accepted.
5. **Payment tracker** — pay-per-short accounting against the editor handoff stream; reconciles what's owed to whom.

**Stack:** Next.js (App Router) · TypeScript · Drizzle ORM · PostgreSQL · NextAuth (Google) · Tiptap · shadcn/ui · TanStack Query · Cloud Run · Firebase Hosting · Terraform · GitHub Actions.
