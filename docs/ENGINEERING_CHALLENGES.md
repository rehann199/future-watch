# Engineering Challenges

## 1. A catalogue that is editable at runtime but never blank

- **Problem:** the studio adds and edits apps often. Rebuilding the site for each edit is slow, but a purely runtime list risks an empty page while loading or when the API fails.
- **Approach:** the Products page ships a bundled list and replaces it with the live registry when it arrives. A completeness check requires every field (name, tagline, descriptions, category, icon, benefits, store links, privacy policy) before an entry is shown; filtering also respects an enabled flag and a per-storefront visibility value.
- **Trade-off:** the bundled list can drift from the live data and must be maintained, and an incomplete entry silently disappears rather than showing an error.
- **Outcome:** immediate first paint, edits without a deploy, and no half-finished cards on the public page.

## 2. Server-side protection for an admin area on a static site

- **Problem:** the admin screens live in the same single-page bundle as the public site, so hiding links is not protection.
- **Approach:** an edge function runs for all paths, returns early for non-admin ones, and for admin paths requires a session cookie derived from the configured password with a keyed hash and compared in constant time. The write APIs verify the same cookie again. If the password is not configured, the area returns a 503 instead of opening up.
- **Trade-off:** a single shared password and a cookie derived from it mean no per-user accounts and no per-session revocation; rotating the password invalidates all sessions.
- **Outcome:** admin routes and write endpoints cannot be used without the secret, while the public site stays fully static.

## 3. Talking to object storage from an edge runtime

- **Problem:** the edge runtime has no S3 SDK, and several handlers need to read and write R2.
- **Approach:** a compact SigV4 signer built on the platform's Web Crypto API, factored into one helper after the first handlers each carried their own copy. A configured-check lets handlers answer clearly when storage is not set up.
- **Trade-off:** hand-rolled signing is code to own and test, but avoids a heavy dependency in a size-constrained runtime.
- **Outcome:** each new store is a few lines on top of shared helpers.

## 4. Stable policy URLs for store listings

- **Problem:** store listings link to privacy URLs that must not change, yet the catalogue is editable and apps are added over time.
- **Approach:** known apps have typed policy entries mapped to fixed routes, with redirects from older paths. Apps added later use a generic route that renders policy HTML from the catalogue record, with an optional custom slug.
- **Trade-off:** two sources of policy text (compiled and stored) can disagree; the page prefers the stored text when a catalogue record has one.
- **Outcome:** existing URLs keep working and new apps need no code change to get a policy page.

## 5. Motion that does not fight the content

- **Problem:** animated sites often feel heavy and can be uncomfortable for motion-sensitive users.
- **Approach:** a small set of shared easing and timing tokens, scroll reveals that run once, hover scaling kept small, and a global setting that follows the operating system's reduced-motion preference.
- **Trade-off:** reveal-on-scroll leaves some sections visually sparse until scrolled into view, which shows up in static captures.
- **Outcome:** consistent, low-key animation across all pages.
