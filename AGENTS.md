# AGENTS.md

## Cursor Cloud specific instructions

This is a minimal single-service **SvelteKit** web app ("Not Workday") — a static page that renders a scrolling marquee. There is no backend, database, or external API dependency.

- **Package manager:** Yarn v1 (`yarn.lock` is the source of truth). The README says `npm install`, but there is no `package-lock.json`; use `yarn` to respect the lockfile.
- **Pinned tooling:** dependencies are 2021-era SvelteKit `1.0.0-next.131` / Vite `2.4.2`. They install and run fine on the VM's Node (v22), but do NOT upgrade them casually — the `next` dist-tag would resolve to incompatible modern builds. The update script uses `--frozen-lockfile` to stay reproducible.
- **Run dev server:** `yarn dev` (serves on `http://localhost:3000`). This is the only service and fully exercises the product.
- **Build:** `yarn build` (Vercel adapter). The build prints harmless warnings (`A11y: Avoid <marquee>` and an unused `h1` CSS selector) — these are expected, not errors.
- **Preview production build:** `yarn preview`.
- **Lint/tests:** none are configured (no lint or test script, no test framework). Prettier is installed and configured in `package.json` if formatting is needed (`npx prettier --check .`).
