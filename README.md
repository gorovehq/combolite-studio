# Combolite Studio (combolite-studio)

`Combolite Studio` is a native desktop database workbench and CMBQL query runner for `combolite` archives, built with GTK 4 and libadwaita via PyGObject. It provides an interactive, visual environment for inspecting, modifying, and querying multi-model database archives (`.cdbarch`):

- **Relational Tables**: Paged data grid with inline cell editing, schema viewer, composite index inspector, and row insert/delete dialogs.
- **Document Pools**: Master-detail document browser, JSON delta revision tree, BM25 full-text search, and live schema validation.
- **Vector Collections**: Exact and approximate k-NN similarity search, distance distribution bars, and an interactive 2D/3D force-directed graph canvas.
- **Graph Links**: Directed, weighted edge inspection, bidirectional link exploration, and real-time path-strength heat maps.
- **Time Series**: Interactive continuous area charts with hover crosshairs, range filters, aggregations (min, max, mean), and point injection.
- **CMBQL Query Console**: Multi-clause query editor with live syntax highlighting, diagnostics squiggles, quick-fix hints, and procedure management.

---

## Prerequisites

### System Requirements
- **Python**: Python >= 3.9
- **GUI Libraries**: GTK 4 (>= 4.10), libadwaita (>= 1.4), PyGObject
- **Optional Tools**: `gtksourceview5` (advanced code editor features), `numpy` (accelerated vector projection and physics math)

### Package Manager Installation

#### Debian / Ubuntu (24.04+)
```sh
sudo apt install python3-gi python3-gi-cairo gir1.2-gtk-4.0 gir1.2-adw-1 gir1.2-gtksource-5 python3-numpy
```

#### Fedora (39+)
```sh
sudo dnf install python3-gobject gtk4 libadwaita gtksourceview5 python3-numpy
```

#### Arch Linux
```sh
sudo pacman -S python-gobject gtk4 libadwaita gtksourceview5 python-numpy
```

---

## Detailed Build & Launch Guide

Combolite Studio communicates with the C99 database engine via `ctypes`. The native shared library (`libcombolite.so`) must be built before launching with the native backend.

### 1. Build the Native Backend Library (from Root)

Use one of the umbrella presets:

```sh
# Build standard CPU-compatible shared library
cmake --preset studio-cpu
cmake --build --preset studio-cpu -j

# Or build with hardware accelerator support
cmake --preset studio-both
cmake --build --preset studio-both -j
```

The compiled library will be placed in `build-studio-cpu/combolite-core/libcombolite.so` (or `build-studio-both/`).

### 2. Run Python Unit Tests

Verify Python backend bindings and CMBQL lexer:

```sh
python3 -m unittest discover -s combolite-studio/tests
```

### 3. Launch Combolite Studio

From the repository root or `combolite-studio/`:

```sh
# Auto-discovers the compiled library in standard build output folders:
python3 -m combolite-studio.combolite_gui

# Or explicitly pass the library path via environment variable:
COMBOLITE_LIB=build-studio-cpu/combolite-core/libcombolite.so python3 -m combolite-studio.combolite_gui
```

### 4. Build Wheel & Install Desktop Integration

```sh
cd combolite-studio
make wheel
pip install dist/combolite_studio-*.whl

# Installs desktop launcher, MIME type definitions, and application icons to ~/.local/
make install-user
```

---

## Getting Started: Exploring a Database

1. **Open a Database**: Click **Open** (or press `Ctrl+O`) and select a `.cdbarch` database archive (such as `combolite-demo/data/gutenfree.cdbarch`).
2. **Explore the Sidebar**: Use `Ctrl+K` to quickly filter stores. Stores are automatically categorized into Tables, Document Pools, Vector Collections, Links, and Time Series.
3. **Inspect Models**:
   - **Tables**: Double-click any cell to edit inline. Click **Insert Row** to append new records.
   - **Documents**: Select a document to view its JSON body, browse previous revisions via the dropdown, or execute BM25 searches.
   - **Vectors & Links**: Click the **Graph** tab to switch to the 2D/3D physics canvas. Drag nodes, scroll to zoom, or orbit in 3D.
4. **Execute CMBQL Queries**: Open the console (`Ctrl+E`), compose multi-line CMBQL queries, and press `Ctrl+Enter` to run.

### Keyboard Shortcuts Reference

| Shortcut | Action |
|---|---|
| `Ctrl+O` | Open database archive |
| `Ctrl+N` | Create new database archive |
| `Ctrl+W` | Close active database |
| `Ctrl+K` | Quick search and filter stores |
| `Ctrl+E` | Focus CMBQL query console |
| `Ctrl+Enter` | Execute active query |
| `Ctrl+.` | Apply all diagnostic quick-fixes |
| `Ctrl+S` | Save current document revision |
| `Ctrl+Shift+T` | Toggle Light / Dark appearance |
| `F5` | Refresh view and reload stores |

---

## Features as Built

- **Full C99 `ctypes` Engine Binding (`NativeBackend`)**: Direct integration with `libcombolite.so` implementing zero-copy buffer allocations and caller-managed memory pools (17/17 tests passing).
- **Graceful Fallback Mode (`MockBackend`)**: Ability to launch and preview the complete UI architecture with synthetic mock stores if no native library is compiled.
- **Interactive 2D / 3D Graph Visualization**: Custom Cairo and vector math engine supporting PCA projections, rubber-band spring physics, 6-hop path-strength Dijkstra heat maps, and depth shading.
- **Integrated Diagnostics & Problem Detection**: Non-blocking background CMBQL analysis providing token-level error squiggles and one-click fixes.
- **Adwaita Responsive Architecture**: Automatic navigation split-view adapting between mobile/narrow window widths and multi-column desktop layouts.

---

## Known Issues

1. **File Dialog File Filter Scope**: The open file dialog filter currently allows legacy `.cmb` and `.db` extensions alongside official `.cdbarch` archives.
2. **GtkSourceView Dependency Fallback**: When running on systems without `gtksourceview5`, the query console gracefully falls back to a standard text view, disabling syntax-highlighting styles.

---

## ToDos

- Add support for exporting query result grids to CSV, JSON, and Parquet formats.
- Implement streaming pagination for table grids with over 100,000 rows.
- Complete implementation of the query **Explain** execution plan visualizer.

---

## Provisioned but Not Yet Implemented

The following capabilities are provisioned in presets, prototypes, or design contracts, but are not yet implemented in the active Python client:

1. **Direct Hardware Accelerator Utilization**:
   - While the `studio-both`, `studio-cuda`, and `studio-vulkan` presets build `libcombolite_accel.so`, Combolite Studio's Python backend currently links only to the core engine library (`libcombolite.so`). Offloading client-side graph layout projections or k-NN distance matrices to GPU backends is provisioned for future releases.
2. **CMBQL Grammar Keyword Sets (`combolite_gui/cmbql.py`)**:
   - The token tables (`KEYWORDS`, `STATEMENT_STARTS`, `FUNCTIONS`) in `cmbql.py` represent provisional placeholder sets awaiting full stabilization of the upstream C99 query compiler.
3. **Remote Serverless Connections**:
   - Studio operates strictly against local on-disk archives (`.cdbarch`); remote networked socket connections or P2P synchronization controls (spec 19) are provisioned but not yet built.

---

## License & Documentation

- Studio Code: MIT License.
- Architectural design specifications and handoff history: [`docs/handoff.md`](docs/handoff.md).
- Reference design mock: `reference/Combolite Studio Preview.dc.html`.
