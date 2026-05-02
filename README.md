# Surface of Revolution CAD Converter

This repository recreates and extends the earlier 2D-to-3D CAD conversion project for the assignment **Surface of Revolution from Closed Two-Dimensional Profiles**.

The program takes a closed 2D profile, validates it, normalizes it into a periodic cubic B-spline on `[0,1]`, revolves it around the `X-axis`, and writes:

- a `STEP` output file
- an `SVG` preview
- optional demo artifacts for inspection

The current practical pipeline is:

```text
input profile -> parse -> validate/repair -> cubic B-spline normalization -> revolve -> STEP + SVG
```

## What This Repository Currently Supports

Implemented today:

- `DXF` input through a lightweight LWPOLYLINE path and a broader `ezdxf` path when available
- `XY` / text / CSV point-set input
- closure repair, self-intersection rejection, axis crossing rejection, winding repair, duplicate-vertex merge
- periodic cubic B-spline normalization on `[0,1]`
- `STEP` export through:
  - `build123d` when installed
  - a faceted fallback writer otherwise
- `SVG` preview export
- demo profile generation
- analytical and edge-case tests

Present but not fully implemented yet:

- `SVG` input parser
- `PNG` input parser
- full PDF-grade DXF support for all entity combinations and planarity/UCS diagnostics

## Repository Layout

```text
revolution/
  cli.py                    command-line entrypoints
  service.py                high-level orchestration
  config.py                 shared tolerances and output settings
  errors.py                 structured exceptions
  demo.py                   demo DXF generation helpers
  parsers/
    dxf_parser.py           DXF ingestion
    xy_parser.py            point-cloud / text ingestion
    svg_parser.py           reserved hook, not yet implemented
    png_parser.py           reserved hook, not yet implemented
  geometry/
    profile.py              profile and spline data models
    repair.py               repair helpers
    validator.py            geometry checks
    spline.py               periodic cubic B-spline normalization
  revolution/
    surface.py              point-based revolution helpers
    solid.py                CAD backend selection
  exporters/
    step_writer.py          STEP output
    svg_writer.py           SVG output
  utils/
    geometry_utils.py       area, centroid, Pappus volume, intersections
  tests/
    test_cli_flow.py
    test_edge_cases.py
    test_parsers_and_cli.py
    test_revolution.py
    test_spline.py
```

## Input Expectations

### DXF

The DXF path expects a single closed 2D profile that is intended to live in the `XY` plane. In the minimal built-in path, it reads an ASCII `LWPOLYLINE`. When `ezdxf` is installed, it can also read a broader subset including `LINE`, `LWPOLYLINE`, and sampled `CIRCLE`.

### XY / CSV / TXT

The `XY` path expects one point per line. Each row should contain two numeric values:

```text
0 10
4 12
4 8
0 10
```

The profile should already describe a single closed loop or be close enough that the closure repair tolerance can snap the endpoints together.

## Output Files

### STEP

The output `STEP` file encodes the revolved geometry. If `build123d` is available, the project exports through that backend. If it is not available, the fallback writer emits a faceted STEP representation from the revolved mesh so the pipeline still produces a CAD exchange file.

### SVG

The output `SVG` includes:

- the closed profile path
- axis annotation
- a latitude-style ellipse for 3D context
- maximum radius and axial extent annotations

## Validation and Repair Rules

The validator currently enforces these behaviors:

- reject profiles with too few points
- merge near-duplicate vertices
- snap small closure gaps within tolerance
- reject large open contours
- reject axis crossing below the `X-axis`
- warn on axis tangency
- reject self-intersection
- reject near-zero enclosed area
- reverse clockwise winding to counter-clockwise

These rules live primarily in [validator.py](</C:/Users/Abhishek/Documents/Codex/2026-04-30/tell-me-about-the-revolution-2d/revolution/geometry/validator.py:1>) and [repair.py](</C:/Users/Abhishek/Documents/Codex/2026-04-30/tell-me-about-the-revolution-2d/revolution/geometry/repair.py:1>).

## Spline Normalization

The spline stage is implemented in [spline.py](</C:/Users/Abhishek/Documents/Codex/2026-04-30/tell-me-about-the-revolution-2d/revolution/geometry/spline.py:1>).

What it does:

- takes the validated closed polygon
- solves for periodic cubic B-spline control points
- creates a periodic spline over `[0,1]`
- exposes point evaluation and derivative evaluation
- samples the spline densely for downstream geometry code

Why it matters:

- it provides a canonical internal representation
- it gives smooth evaluation and smooth first derivatives
- it aligns much more closely with the assignment’s requested internal model than a raw polygon alone

## Quick Start

From the repository root:

```powershell
python -m pip install -r requirements.txt
python -m pip install -e .
python -m revolution --help
```

Generate a demo:

```powershell
python -m revolution make-demo --out demo_profile.dxf --shape amoeba
```

Convert the demo:

```powershell
python -m revolution convert demo_profile.dxf --step-out demo_output.step --svg-out demo_output.svg
```

## Running Tests

```powershell
python -m pytest
```

Current test status in this repository:

- `24 passed`
- `90%` coverage

## Demo Artifacts

The repository root already contains generated demo files such as:

- `demo_profile.dxf`
- `demo_profile_preview.svg`
- `demo_profile_preview.jpg`
- `demo_output.step`
- `demo_output.svg`
- `demo_output.jpg`

It also contains preserved shape variants:

- `demo_profile_amoeba.dxf`
- `demo_profile_star.dxf`
- `demo_profile_circle.dxf`

## Current Alignment With the PDF

This repository now does a much better job on:

- periodic cubic B-spline normalization
- edge-case tests
- analytical checks
- structured validation flow

It still does **not** fully satisfy the full PDF on every point. The biggest remaining gaps are:

- missing real `SVG` input parsing
- missing real `PNG` input parsing
- incomplete DXF entity coverage relative to the PDF
- no full CAD-kernel-based validity validation in the fallback environment

## Report

The detailed submission-style report is here:

[PROJECT_REPORT.md](</C:/Users/Abhishek/Documents/Codex/2026-04-30/tell-me-about-the-revolution-2d/PROJECT_REPORT.md:1>)
