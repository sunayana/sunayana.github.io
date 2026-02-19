+++
title = "Global Flood Risk Dataset at 90m Resolution"
date = 2023-06-01
weight = 1
draft = false
tags = ["Climate Tech", "Flood Modeling", "Julia", "Python", "Big Data", "Geospatial"]
categories = ["Climate Tech"]
description = "Co-architected a globally-scoped riverine and pluvial flood risk system producing 90m resolution inundation maps"
+++

## Cervest - Climate Intelligence Platform

**Duration**: June 2021 - June 2023
**Location**: Berlin, Germany

### Why This Matters

Flood risk is one of the most financially significant climate hazards, yet most available flood data is either too coarse for asset-level decisions or too expensive to produce at global scale. This project tackled that gap — delivering a high-resolution, multi-scenario flood risk dataset that enables insurers, asset managers, and governments to quantify flood exposure anywhere on Earth.

### Impact

- Delivered a **global 90 m resolution flood risk dataset** covering both riverine and pluvial flooding
- Multi-scenario coverage: return periods from 2 to 1000 years across historical and RCP climate scenarios
- Used commercially for climate risk assessment
- Provided foundation for Cervest's climate intelligence platform

### Project Overview

Co-architected and built a **globally-scoped riverine and pluvial flood risk system** that produces 90 m resolution inundation maps across multiple climate scenarios and return periods. The system combines physics-based hydraulic modelling, Bayesian statistical inference, and high-performance geospatial downscaling, orchestrated at scale via Argo Workflows on Kubernetes.

---

## Technical Deep Dive

### Riverine Flooding

The riverine pipeline computes flood risk end-to-end — from raw climate model discharge data to actionable inundation depth maps:

**Bathtub Flood Inundation Model (Julia)** — I built the core algorithm that converts flood *volumes* into flood *depths* across a landscape. Given a known flood volume for a river reach, the model fills the surrounding floodplain from the lowest elevation upward using the **HAND (Height Above Nearest Drainage)** metric as a proxy for relative elevation above the river channel. The model classifies pixels by flood susceptibility and integrates with global water masks for accurate inundation mapping.

**Multi-Resolution Downscaling with Volume Conservation** — Spatial downscaling from coarse climate model outputs to 90 m inundation depth maps, with volume conservation accounting for latitude-dependent pixel area. The approach captures fine-scale terrain features such as levees and embankments.

**Hydrological Network Analysis** — Strahler stream order extraction, HAND computation, and sub-basin delineation using `pyflwdir`. Long basins are cut at intermediate outlets to keep computational units memory-efficient while preserving hydrologic coherence. Basin edge masking removes incomplete basins at tile boundaries to prevent artefacts.

**Bayesian Bias Correction** — ISIMIP climate model discharge projections are bias-corrected against historical GLOFAS observations using Stan-based Bayesian quantile mapping. Gumbel (extreme value) distributions are fitted to annual maximum discharge series via MCMC inference, producing uncertainty-quantified return levels at the 5th, 50th, and 95th percentiles.

**Physics-Based Hydraulic Modelling** — Flood volumes computed using Manning's equation for open-channel flow, with empirical relationships for river geometry (width, depth, slope) derived from upstream catchment area. The model accounts for channel vs. floodplain friction, river sinuosity, and Strahler stream order.

---

### Pluvial Flooding

#### RichDEM.jl — Julia Bindings for Fill-Spill-Merge

![Fill-Spill-Merge: routing water through depressions](fill-spill-merge.png "Fill-Spill-Merge algorithm, inspired by Barnes et al. 2021")

I created **RichDEM.jl**, Julia language bindings for the C++ [RichDEM](https://github.com/Cervest/richdem) library, using CxxWrap.jl. This made the high-performance Fill-Spill-Merge algorithm and core data structures available in Julia for pluvial flood modelling:

- Wrapped C++ classes (Array2D, DepressionHierarchy) with proper template handling for float/double types
- Exposed the BucketFillFromEdges algorithm to Julia
- Built the RichDEM_jll binary dependency and the RichDEM.jl package
- Set up a containerised development environment with Docker and VSCode integration
- Fixed bugs in the core C++ library (indexing, data type handling)

This enabled depression hierarchy computation from elevation data, which fed into rainfall-scenario-driven pluvial inundation modelling with serialised (compressed JSON) depression data stored on S3.

---

### Performance Engineering for Global Scale

Making these pipelines process the entire globe required significant performance work:

- **DiskArrays broadcasting** — Dramatically reduced raster computation times by leveraging lazy broadcasting over memory-mapped arrays
- **Memory-mapped raster access** — Enabled processing of very large tiles without loading them entirely into RAM
- **Zstandard compression** — Balanced I/O speed and storage for serialised geospatial arrays
- **StructArrays** — Cache-friendly, column-oriented storage for lookup tables
- **Unitful.jl** — Compile-time dimensional analysis preventing unit mismatch bugs
- **Idempotent task generation** — Workflows can be safely restarted after Kubernetes evictions
- **Resource tuning** — Iterative optimisation of memory limits, parallelism, and retry strategies for cloud-scale processing

---

### Engineering

#### Dual-Language Architecture

| Language   | Role                                                         |
| ---------- | ------------------------------------------------------------ |
| **Python** | Data ingestion, statistical modelling, orchestration, export |
| **Julia**  | High-performance geospatial downscaling, bathtub flood model |

#### Migration from Pachyderm to Argo Workflows

Led the migration of the entire pipeline from Pachyderm to Argo Workflows on Kubernetes:

- Rewrote task generation for Argo's parallelism patterns
- Created workflow definitions for all pipeline stages
- Introduced idempotent reruns with success-tracking directories
- Highly parallel workflows with retry strategies

#### Infrastructure

- Multi-stage Docker builds for Python and Julia containers
- CI/CD with GitHub Actions (black, isort, flake8, mypy, JuliaFormatter)
- Pre-commit hooks for both languages
- S3-based data storage with Zarr and Parquet export
- H3 hexagonal indexing for downstream query APIs

#### Technical Stack

**Languages**: Julia, Python

**Geospatial**: ArchGDAL, pyflwdir, whitebox-tools, GDAL, Rasterio

**Statistical**: CmdStanPy, Stan (Bayesian MCMC)

**Data Formats**: GeoTIFF, Zarr, Parquet, NetCDF

**Infrastructure**: Docker, Argo Workflows, Kubernetes, AWS S3

**Julia Libraries**: DiskArrays, StructArrays, Unitful.jl, CxxWrap.jl

---

**Technologies**: Julia, Python, Stan, Argo Workflows, Kubernetes, Docker, Geospatial Processing

**Domain**: Climate tech, Flood hydrology, Extreme value statistics, Distributed computing
