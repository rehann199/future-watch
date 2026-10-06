# Technical Case Study

## Summary

Future Watch is a single-page React application hosted on Netlify. A small set of Deno edge functions provide a runtime API and a password gate, and Cloudflare R2 stores JSON documents that the public catalogue reads. The visitor-facing site is fully static; only the catalogue data and the admin area depend on the edge layer.

## Goals

- Present the studio and its apps clearly on every screen size.
- Let the team add or edit an app without rebuilding and redeploying the site.
- Serve stable per-app privacy-policy URLs for store listings.
- Keep an internal admin area off the public internet without running a separate server.

## Stack

React 18 and TypeScript on Vite 5 (SWC), React Router 6, Tailwind CSS 3 with Radix-based components, Framer Motion, TanStack Query, EmailJS. Hosting on Netlify with edge functions in Deno; storage on Cloudflare R2.

## How it fits together

1. **Static shell.** `vite build` emits a single-page app. A catch-all redirect rewrites unknown paths to the shell so client-side routes resolve on refresh.
2. **Route-level splitting.** Every page is a lazy-loaded route behind a loading fallback. The production build emits one small chunk per public page (the Products chunk is about 34 kB, 11 kB gzipped; the entry chunk is about 485 kB, 142 kB gzipped).
3. **Runtime catalogue.** The Products page queries `/api/apps`. An edge function reads one JSON document from R2 and returns it. The page keeps a bundled fallback list so the first paint never waits on the network.
4. **Completeness gate.** A record is listed only when every required field is present; otherwise it is hidden. This stops half-edited entries from reaching the public page.
5. **Policies.** Seventeen policies are compiled into the app as typed entries and mapped to stable routes. A generic `/privacy/:appId` route renders policy HTML stored with a catalogue record, so a new app gets a policy page without a code change.
6. **Admin.** The admin area is part of the same bundle but every `/admin` request passes through an edge function that checks a session cookie before the shell or any admin API responds.

## Notable implementation choices

- **No server framework.** Edge functions are small, single-purpose handlers (catalogue, icon upload, ad-verification file, build status, repository records, widget layout, admin gate).
- **Own request signing for R2.** The edge runtime has no S3 client, so a short SigV4 signer is shared by the handlers.
- **Environment-only secrets.** Credentials and the admin password are read from the host environment in the edge runtime. The contact form's public service identifiers come from build-time variables.
- **Dev parity.** A Vite dev plugin mirrors the catalogue endpoint so `vite` shows real data without running the edge runtime.

## Performance and accessibility (verified)

- Lazy-loaded routes, with an animation library chunk split from page code.
- Reduced-motion preference honoured globally.
- Navigation toggle exposes `aria-label` and `aria-expanded`.
- No horizontal overflow at 412 px.
- Decorative glow layers are marked `aria-hidden`.

## SEO (verified)

- Static title, description, Open Graph and Twitter tags in the HTML shell.
- `sitemap.xml` and `robots.txt` (admin disallowed).
- Per-app policy pages set their own document title.
- Page metadata is defined once in the HTML shell, which suits a client-rendered single-page app.

## What is not covered

Admin screens, analytics dashboards and build-tracking views are deliberately left out of screenshots. Marketing statements on the live site are the studio's claims and are not treated as measured facts here.
