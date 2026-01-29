# Product: PCB Renderer CLI

## Why This Project Exists

This project implements a **PCB (Printed Circuit Board) Renderer CLI** tool designed as a solution to the Quilter Backend Engineer Code Challenge. The tool bridges the gap between raw ECAD (Electronic Computer-Aided Design) data and visual verification, providing engineers with a lightweight, focused utility for rendering and validating PCB designs.

## Problems It Solves

1. **Visualization Gap**: ECAD tools export board designs as complex JSON files that are not human-readable. This tool converts them into visual formats (SVG, PNG, PDF) for review and documentation.

2. **Design Validation**: PCB designs can contain errors that are difficult to detect manually. The tool validates against 14 specific error classes including:
   - Missing or malformed board boundaries
   - Invalid component placement (outside board edges)
   - Self-intersecting polygons
   - Invalid via geometry (holes larger than outer diameter)
   - Dangling traces referencing non-existent nets or layers
   - Negative or zero-width traces

3. **Cross-Platform Consistency**: Provides deterministic, offline rendering that produces identical outputs across different platforms, suitable for automated testing and version control.

4. **Engineering Review**: Enables quick visual verification of board layouts, component placement, trace routing, and keepout regions without requiring heavy CAD software.

## How It Should Work

### Core Workflow
1. **Parse**: Load ECAD JSON files and normalize all units to millimeters
2. **Validate**: Check board integrity against 14 error categories, collecting all errors rather than failing fast
3. **Transform**: Apply coordinate system conversions (ECAD to SVG space), component rotations, and mirroring for back-side components
4. **Render**: Generate layered visualizations using Matplotlib with proper Z-ordering
5. **Export**: Output to SVG, PNG, or PDF formats via unified savefig() API

### CLI Interface
```bash
pcb-render validate <board.json>          # Validate without rendering
pcb-render render <board.json> -o out.svg # Render to SVG
pcb-render render <board.json> --layers TOP,BOTTOM --format png
pcb-render info <board.json>              # Display board metadata
```

### Rendering Features
- **Board Boundary**: Outline/cutout rendering
- **Layer Visualization**: Color-coded layers (Red=Top, Blue=Bottom, Inner=Green/Purple)
- **Components**: Bounding boxes with reference designators (RefDes) and halos for readability
- **Traces**: Width-mapped copper paths
- **Vias**: Drilled holes with annular rings
- **Keepouts**: Restricted areas with hatching patterns (///) and warning colors

## User Experience Goals

1. **Correctness First**: All outputs are deterministic and mathematically correct; validation is comprehensive
2. **20-Minute Review**: Codebase designed to be reviewable within 20 minutes with clear separation of concerns
3. **Minimal Dependencies**: Uses only Python 3.11+, Pydantic v2, Matplotlib, and NumPy for fast setup
4. **Clear Error Reporting**: Structured error objects with codes, severity levels, messages, and JSON paths
5. **Docker Support**: Containerized execution for consistent environments
6. **Testability**: Pure functions, immutable data flows, and comprehensive test coverage with pytest, Hypothesis, and Syrupy snapshot testing

## Current Status

**Planning Phase**: The project has extensive documentation in `plan_docs/` including architecture guides, implementation guides, and 26 sample board JSON files. No source code has been implemented yet.

## Secondary Purpose

This repository also serves as an **AI-powered template system** for AI-assisted application development, featuring:
- Remote canonical instruction modules from `nam20485/agent-instructions`
- Local AI instruction modules for workspace-specific configuration
- Dynamic workflow orchestration system
- GitHub automation scripts and security scanning (TruffleHog)
- Docker-based workflow agent for isolated execution
