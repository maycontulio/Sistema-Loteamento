# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sistema Loteamento is a browser-based tool for managing Brazilian land subdivisions (loteamentos). It is a collection of standalone HTML files — no build step, no server, no dependencies. Open any `.html` file directly in a browser.

The HTML files live one level up, in `../` (the Downloads folder). The latest version is `../loteamento_multiselect.html`.

## Running the App

Open any HTML file directly in a browser (double-click or drag into browser). No server required — all logic is client-side. Background image and lot data persist via `localStorage`.

## File History

Each file represents a self-contained iteration:

| File | Purpose |
|---|---|
| `loteamento.html` | Read-only interactive viewer (original) |
| `loteamento_editor*.html` | Early editor versions |
| `loteamento_manual.html` | Manual polygon drawing variant |
| `loteamento_multiselect.html` | Multi-selection batch edit |
| `loteamento_pro.html` | Advanced pre-v2 version |
| `loteamento_v2.html` → `loteamento_v5_1.html` | Progressive main versions |

**Active development target**: `loteamento_multiselect.html`

## Architecture

Each HTML file is fully self-contained (CSS + JS inline, no external JS dependencies).

### Layout Structure

```
Fixed toolbar (52px, z-index 300)
├── Mode buttons: ＋ Novo Lote | ⊞ Criar Vários | ✎ Editar | 👁 Visualizar
├── Image upload button
├── 📐 Escala (pixel/meter calibration)
└── Export: ⬇ JSON | ⬇ HTML | 🗑 Limpar

Scrollable map area (#mw, right edge = sidebar width)
├── #canvas (sized to background image, default 1494×842px)
│   ├── #bgimg (background image, pointer-events:none)
│   ├── #gridovl (CSS grid overlay, decorative)
│   ├── #svg (SVG layer — all lotes rendered here)
│   └── #selbox (rubber-band selection rectangle)
└── #dz (drop zone, shown when no data loaded)

Fixed sidebar (#sb, 316px, right edge)
├── Lote list (grouped by quadra) OR single lote form OR multi-select panel
└── Stats bar: Disponíveis | Vendidos | Reservados counts
```

### Data Model

```js
// Top-level state
let lotes = [];          // flat array of all Lote objects
let mode = 'view' | 'edit' | 'create';
let selId = null;        // single selected lote id
let selIds = new Set();  // multi-selected lote ids (rubber-band or Shift+click)
let nextId = 1;          // incrementing id counter

// Lote object
{
  id: string,            // "l1", "l2", ...
  quadra: number,        // quadra number (display grouping only)
  numero: number,        // lot number within quadra
  tamanho: number,       // m² (user-entered)
  preco: number,         // R$
  status: 'disponivel' | 'vendido' | 'reservado',
  x, y: number,          // top-left corner in canvas coordinates (px)
  w, h: number,          // width and height in canvas pixels
  angle: number,         // rotation in degrees
}

// Scale system
let escalaPxM = 0;       // pixels per meter (0 = no scale calibrated)
let scaleMeds = [];      // array of {px, m} calibration points
```

### Key Functions

| Function | Role |
|---|---|
| `render()` | Redraws entire SVG — call after any state change |
| `criarLoteEl(l)` | SVG factory for a single lot element (rect + event handlers) |
| `addLabels(g, l)` | Adds number and m² text labels inside a lot's SVG group |
| `addHandles(g, l)` | Adds resize/rotate handles in edit mode |
| `renderSB(view, payload)` | Swaps sidebar: `'l'` = single lot form, `'multi'` = batch panel, else lot list |
| `setMode(m)` | Switches view/edit/create mode, updates cursor and toolbar |
| `abrirPopup(e, l)` | Shows floating info popup on lot click in view mode |
| `placeLot(x, y)` | Creates a single new lot at canvas coords and enters edit mode |
| `criarBulk(cfg)` | Batch-creates a grid of lots from modal config |
| `abrirModalBulk()` | Opens the "Criar Vários" bulk creation modal |
| `salvarStorage()` / `carregarStorage()` | Persist/restore `lotes` to `localStorage['lot_man_v1']` |
| `expJSON()` | Download `loteamento.json` (raw lotes array) |
| `expHTML()` | Generate and download a standalone read-only viewer HTML |
| `abrirModalEscala()` | Opens scale calibration dialog |
| `iniciarMedicao()` | Starts two-point pixel/meter measurement mode |

### CSS Design System

CSS custom properties on `:root` — always use these, never hardcode colors:

- `--bg / --s1 / --s2 / --s3`: dark navy background layers
- `--gn / --gna / --gnh / --gns`: green (disponivel)
- `--rd / --rda / --rdh / --rds`: red (vendido)
- `--yw / --ywa / --ywh / --yws`: yellow (reservado)
- `--ac / --ac2`: blue accent
- `--tx / --mu / --mu2`: text (main / muted / muted-light)
- `--tb: 52px` / `--sb: 316px`: toolbar height / sidebar width
- `--font: 'Space Grotesk'` / `--mono: 'JetBrains Mono'`

### Persistence

- `localStorage['lot_man_v1']` — JSON blob `{lotes, nextId}`
- `localStorage['lot_bg']` — base64 background image (can be large)
- `localStorage['lot_scale_v1']` — JSON array of `{px, m}` calibration pairs
- Export to JSON or self-contained HTML viewer for sharing

## Lot Status Colors

| Status | PT-BR label | Color var |
|---|---|---|
| `disponivel` | Disponível | `--gn` (green) |
| `vendido` | Vendido | `--rd` (red) |
| `reservado` | Reservado | `--yw` (yellow) |

## Constants

```js
const CW = 1494, CH = 842;  // default canvas size (px)
const MIN_W = 18, MIN_H = 14; // minimum lot dimensions (px) for resize handles
const SNAP_DIST = 7, SNAP_PROX = 90; // snap-to-edge parameters
```

## Supporting Documents

The `../Contratos Loteamentos/` folder contains contract templates (`.doc`/`.docx`) and `../Loteamento Santo Antonio - Vendas*.pdf` files are sales maps for the real-world project this tool was built for.
