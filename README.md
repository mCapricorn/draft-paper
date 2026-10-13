# Draft Paper

[中文](README.zh-CN.md) | English

Draft Paper is a lightweight single-user online sketch/draft board built as a Vite + Vue app. The repository is intentionally front-end only: it stores draft data in the browser, supports multiple local drafts, and keeps the editing flow fully local without a backend or account system.

This documentation matches the current implementation in the repo rather than an older prototype description.

## What the app does now

- Workbench with a list of recent drafts and a way to create or resume local drafts
- Multiple draft records, each with its own autosave key in browser storage
- Import and export of draft files (`.zxkjbb` and legacy `.json` data)
- Whiteboard editing with drawing, erasing, pan, zoom, rotate, undo/redo, and screenshot capture
- Performance mode switch for high-volume handwriting or large-area sketching
- Browser locale selection (`zh-CN` / `en-US`) and local persistence of app state

## Actual repo workflow

The project is configured as a Vite app in `package.json` and `vite.config.js`:

- `npm install`
- `npm run dev` starts the dev server
- `npm run build` creates a production build in `dist/`
- `npm run preview` serves the built app locally
- The dev server is configured for `host: '0.0.0.0'` and port `5173`

This reflects the repo’s current scripts and config; there is no separate backend, deploy script, or cloud-sync service in the project.

## Quick start

```bash
npm install
npm run dev
```

Then open the app in the browser at:

- `http://localhost:5173`

## Production build

```bash
npm run build
```

Optional local preview after building:

```bash
npm run preview
```

## Data and storage model

The app is intentionally local-only:

- App and draft metadata are stored in `localStorage`
- Each draft has its own autosave key under the browser storage, managed from `src/App.vue`
- The board state and export/import logic are handled in `src/components/WhiteboardView.vue`
- Drafts are not synced to a remote server or user account

If you want long-term backup, export the board as a `.zxkjbb` file from the editor UI. Imported files may be `.zxkjbb` or older JSON draft data.

## Suggested development workflow

1. Install dependencies with `npm install`.
2. Start the app with `npm run dev` for active development.
3. Validate changes with `npm run build` before finalizing a version.
4. Use `npm run preview` only to inspect the built bundle locally.

## Repository structure

- `src/App.vue` — workbench, recent drafts, locale toggle, draft state management
- `src/components/WhiteboardView.vue` — board interactions, drawing, export/import, autosave, screenshot, performance settings
- `vite.config.js` — Vite dev server configuration
- `index.html` — app entry HTML
- `package.json` — scripts and dependencies for the project

## Project scope

This repository focuses on a single-user local sketching workflow. It does not include multi-user collaboration, cloud sync, or accounts.

## Language

- [中文](README.zh-CN.md)
- English
