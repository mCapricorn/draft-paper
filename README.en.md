# Draft Paper

This is an independent online draft paper project designed to provide a lightweight, publicly shareable, single-user electronic whiteboard / sketch notebook.

The current version has evolved from a "single-page whiteboard prototype" to a "single-user drafting application with a workspace":

- Create, continue, and manage multiple local drafts from the workspace
- Each draft has its own local autosave storage key
- Supports migration from the legacy single-draft local cache to the new multi-draft structure
- Preserves whiteboard autosave, screenshots, import/export, and shape optimization features

## Project positioning

- No multi-user collaboration features
- Focused on single-user online drafting
- Keeps it simple, stable, and directly usable for personal notes, quick ideation, and handwritten sketches

## Completed transformations

- Extracted the board core code from the main project while keeping it independently runnable
- Created a public GitHub repository: https://github.com/mCapricorn/draft-paper
- Added draft naming and performance mode entry points for personal use scenarios
- Preserved local recovery, import/export, and screenshot capabilities

## Features

- Workspace homepage and recent drafts list
- Multi-draft local management
- Canvas drawing and erasing
- Zoom / pan / rotate
- Undo and redo
- Handwriting recognition and shape optimization
- Import and export
- Auto-save
- Screenshot export
- Performance mode: reduces unnecessary redraws during large-area writing and decreases seam-detail processing

## Usage

1. Open the app and click "New Draft" on the workspace to start writing.
2. Content is automatically saved to browser local storage while editing.
3. Return to the workspace and continue any draft from the "Recent Drafts" list.
4. For long-term backups, use the export feature on the right side of the whiteboard to save a `.zxkjbb` file.
5. If you previously used the old single-draft cache, the workspace provides a "Restore Old Version Local Drafts" entry.

## Quick Start

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
```

## Performance notes

- Coalesced redraw and offscreen rendering layers reduce wasted redraws during high-frequency input.
- In performance mode, the shape simplification threshold and seam detail are reduced to improve responsiveness during large-area writing.
- This version focuses on the single-user drafting experience and does not introduce multi-user collaboration or real-time sync.

## Data scope

- This version is frontend-only and local-only; it does not include accounts, cloud sync, or multi-user collaboration.
- Draft data is stored in the current browser's local storage by default. Clearing the browser site data will remove it.
- For long-term retention, export drafts regularly.
