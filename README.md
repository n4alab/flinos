# FLiNOS

<p align="center">
  <img src="img/flinos_denim_dark_on_black.svg" alt="FLiNOS — Friendly Linux Network Operating System" width="640">
</p>

> **Status: in development.** FLiNOS is not yet ready for production use.

FLiNOS (**Friendly Linux Network Operating System**) is a Debian-based network
appliance designed to feel like a data-center switch. It provides Layer 2 and
Layer 3 networking in a purpose-built operating environment, while remaining
able to operate as a router when a deployment requires it.

## Networking model

FLiNOS focuses on practical switching and routing workflows:

- Layer 2 networking, including interface management, VLANs, and VLAN-aware
  bridging.
- Layer 3 addressing and routing for routed network deployments.
- A Junos-style command-line interface with a candidate configuration,
  comparison, commit, rollback, and verification workflow.

## This repository

This is the public distribution repository for FLiNOS. It contains published
documentation, built release artifacts, and project images; it does **not**
contain FLiNOS source code.

Future releases may include the following generated artifacts:

- `flinos-<release>.qcow2` — virtual appliance image.
- `flinos-<release>.json` — release metadata.
- `flinos-<release>.version` — release version information.

## Branding assets

The `img/` directory contains editable SVG wordmarks and icons. The branding
palette is denim (`#315A91`), dark denim (`#1F3F6B`), near black (`#05070A`),
and white (`#F5F7FA`). The SVG set is also provided as
`flinos_svg_set.zip`.
