# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sistema Loteamento is a browser-based tool for managing Brazilian land subdivisions (loteamentos). It is a collection of standalone HTML files — no build step, no server, no dependencies. Open any `.html` file directly in a browser.

The HTML files live one level up, in `../` (the Downloads folder). The latest version is `../loteamento_v5_1.html`.

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

**Active development target**: `loteamento_v5_1.html`

## Architecture

Each HTML file is fully self-contained (CSS + JS inline, no external JS dependencies).

### Layout Structure (v5+)

```
Fixed toolbar (54px, z-index 300)
├── Mode buttons: Nova Quadra | Editar | Visualizar
├── Draw controls: Fechar | Cancelar (hidden unless drawing)
├── Image upload button
└── Export: ⬇ JSON | ⬇ HTML | 🗑 Limpar

Scrollable map area (#mw, right edge = sidebar width)
├── #canvas (sized to background image, default 1494×842px)
│   ├── #bgimg (background image, pointer-events:none)
│   ├── #gridovl (CSS grid overlay, decorative)
│   └── #svg (SVG layer — all quadras and lotes rendered here)
└── #dz (drop zone, shown when no data loaded)

Fixed sidebar (#sb, 320px, right edge)
├── Quadras list OR Lote detail form (swapped by renderSB())
└── Stats bar: Disponíveis | Vendidos | Reservados counts
```

### Data Model

```js
// Top-level state
let quadras = [];   // array of Quadra objects
let mode = 'quadra' | 'edit' | 'view';

// Quadra object
{
  id: string,           // "Q1", "Q2", ...
  nome: string,         // display name, e.g. "Quadra A"
  pontos: [{x, y}],     // polygon vertices in SVG/canvas coordinates
  areaTotal: number,    // m²
  preco: number,        // R$ per lot
  lotes: [Lote]
}

// Lote object
{
  id: string,           // "L1", "L2", ...
  x, y, w, h: number,  // bounding rect in canvas coordinates
  status: 'disponivel' | 'vendido' | 'reservado',
  area: number,         // m²
  dimW, dimH: number,   // meters
  preco: number         // R$
}
```

### Key Functions

| Function | Role |
|---|---|
| `render()` | Redraws entire SVG — call after any state change |
| `gerarLotes(q, numLotes, areaTotal, preco, status)` | Creates lots grid inside a quadra polygon |
| `solveGrid(n, bW, bH)` | Finds optimal cols×rows grid for N lots in a bounding box |
| `criarLoteEl(q, l)` | SVG factory for a single lot element |
| `renderSB(view, payload)` | Swaps sidebar between quadra list and lot detail |
| `setMode(m)` | Switches draw/edit/view mode, updates cursor and toolbar |
| `salvarStorage()` / `carregarStorage()` | Persist/restore `quadras` to `localStorage['lot_v5']` |
| `expJSON()` | Download `loteamento.json` (raw quadras array) |
| `expHTML()` | Generate and download a standalone read-only viewer HTML |

### CSS Design System

CSS custom properties on `:root` — always use these, never hardcode colors:

- `--bg / --s1 / --s2 / --s3`: dark navy background layers
- `--gn / --gna / --gnH / --gns`: green (disponivel)
- `--rd / --rda / --rdH / --rds`: red (vendido)
- `--yw / --ywa / --ywH / --yws`: yellow (reservado)
- `--ac / --ac2`: blue accent
- `--tx / --mu / --mu2`: text (main / muted / muted-light)
- `--tb: 54px` / `--sb: 320px`: toolbar height / sidebar width
- `--font: 'Space Grotesk'` / `--mono: 'JetBrains Mono'`

### Persistence

- `localStorage['lot_v5']` — JSON blob `{quadras, nQId, nLId}`
- `localStorage['lot_bg']` — base64 background image (can be large)
- Export to JSON or self-contained HTML viewer for sharing

## Lot Status Colors

| Status | PT-BR label | Color var |
|---|---|---|
| `disponivel` | Disponível | `--gn` (green) |
| `vendido` | Vendido | `--rd` (red) |
| `reservado` | Reservado | `--yw` (yellow) |

## Legal Constants (v5)

```js
const AREA_MIN  = 125;  // m² — minimum lot area (Brazilian urban parceling law)
const FRONT_MIN = 10;   // m  — minimum lot frontage
const CW = 1494, CH = 842;  // default canvas size (px)
```

## Supporting Documents

The `../Contratos Loteamentos/` folder contains contract templates (`.doc`/`.docx`) and `../Loteamento Santo Antonio - Vendas*.pdf` files are sales maps for the real-world project this tool was built for.
