# Architecture

## System Architecture

The PCB Renderer CLI follows a layered architecture designed around correctness, testability, and clear separation of concerns.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│ Presentation Layer (CLI)                                │
│  - cli.py: argument parsing, error display, progress    │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│ Application Layer (Orchestration)                       │
│  - parse.py: JSON loading, unit normalization           │
│  - validate.py: board validation rules                  │
│  - transform.py: coordinate conversions                 │
│  - render_svg.py: Matplotlib rendering pipeline         │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│ Domain Layer (Core Business Logic)                      │
│  - models.py: Pydantic data models                      │
│  - geometry.py: geometric primitives and operations     │
│  - errors.py: structured error types                    │
└─────────────────────────────────────────────────────────┘
```

## Source Code Paths (Planned)

**Package Structure**:
```
pcb_renderer/
├── __init__.py           # Package initialization
├── cli.py               # CLI entry point (argparse or Typer)
├── models.py            # Pydantic data models
├── parse.py             # JSON parsing and unit normalization
├── validate.py          # Validation rules
├── transform.py         # Coordinate transforms
├── render_svg.py        # Matplotlib rendering
├── geometry.py          # Geometric utilities
└── errors.py            # Error types and reporting
```

**Tests Structure**:
```
tests/
├── __init__.py
├── conftest.py          # Pytest fixtures
├── test_parse.py
├── test_validate.py
├── test_transform.py
├── test_render.py
├── test_geometry.py
├── invalid_boards/      # Test fixtures for invalid boards
└── snapshots/           # Syrupy snapshot files
```

## Key Technical Decisions

### Technology Stack
- **Python 3.11+**: Modern Python with improved error messages and performance
- **Pydantic v2**: Data validation and serialization with significant performance gains
- **Matplotlib**: Single rendering dependency supporting SVG, PNG, and PDF via `savefig()`
- **NumPy**: Optional acceleration for geometric transforms
- **uv**: Fast Python package manager with lockfile support

### Coordinate System Handling
- **ECAD Coordinates**: Right-handed, origin at lower-left, Y increases upward
- **SVG Coordinates**: Origin at top-left, Y increases downward
- **Transform Pipeline**: Single Y-axis inversion at render time via `transform.py`

### Validation Strategy
- **Collector Pattern**: Accumulate all errors rather than failing fast
- **Three Validation Levels**:
  1. Field-level: Pydantic validators for type/range constraints
  2. Model-level: Cross-field constraints within single models
  3. Board-level: Global constraints across all models

### Rendering Z-Order
Lower values appear beneath higher values:
1. Board outline (z=1)
2. Copper pours (z=2)
3. Traces (z=3)
4. Vias (z=4)
5. Components (z=5)
6. Reference designators (z=6)
7. Keepouts (z=7)

## Design Patterns

### Immutable Data Flow
- Pydantic models are frozen after creation
- Transform functions return new objects rather than mutating
- Enables predictable testing and prevents side effects

### Pure Functions
- Geometric operations are pure functions
- No global state dependencies
- Easy to unit test with explicit inputs/outputs

### Error Accumulation
- Validation functions return lists of structured errors
- Error type: `{code, severity, message, json_path}`
- 14 defined error codes covering all validation scenarios

## Component Relationships

### Data Flow
```
JSON File → parse.py → models.py → validate.py → transform.py → render_svg.py → Output
                │            │           │            │              │
                └────────────┴───────────┴────────────┴──────────────┘
                              Error Collection
```

### Model Dependencies
```
Board (root)
├── Boundary: Polygon
├── Stackup: List[Layer]
├── Nets: List[str]
├── Components: Dict[str, Component]
│   ├── Transform (position, rotation, side)
│   ├── Outline: Polygon
│   └── Pins: Dict[str, Pin]
│       └── Pad geometry, net reference
├── Traces: Dict[str, Trace]
│   ├── Path: Polyline
│   ├── Width: float
│   └── Layer/Net references
├── Vias: Dict[str, Via]
│   ├── Center: Point
│   ├── Diameters
│   └── Layer span
├── Pours: Dict[str, Pour]
└── Keepouts: List[Keepout]
    └── Polygon with optional layer scope
```

## Critical Implementation Paths

### 1. Parsing Pipeline
- Load JSON → Detect coordinate format → Normalize units to mm
- Handle both flat `[x1,y1,x2,y2]` and nested `[[x1,y1],[x2,y2]]` coordinate arrays
- Default to microns if units unspecified

### 2. Transform Pipeline
- Component placement: translate → rotate → mirror (for BACK side)
- Coordinate conversion: ECAD to SVG via Y-axis inversion
- ViewBox calculation: board boundary + 5-10% padding

### 3. Rendering Pipeline
- Setup matplotlib Figure with equal aspect ratio
- Draw elements in Z-order sequence
- Apply layer colors (Red=TOP, Blue=BOTTOM, Green/Purple=Inner)
- Save via `savefig()` to SVG/PNG/PDF

### 4. Validation Pipeline
- Field validation via Pydantic
- Geometric checks (self-intersection, containment)
- Reference validation (layers, nets exist)
- Return all errors as structured list
