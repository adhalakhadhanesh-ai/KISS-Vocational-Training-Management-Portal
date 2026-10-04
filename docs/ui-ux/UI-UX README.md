# KISS VMP — UI/UX Reference

## Overview

**KISS VMP (KISS Vocational Management Portal)** is an institutional, ERP-style web portal that manages the vocational training lifecycle for three roles: **Student**, **Teacher** and **Admin**.

This folder holds the UI/UX documentation for the project:

| Item | Purpose |
|---|---|
| [`UI_UX_SPEC.md`](./UI_UX_SPEC.md) | Detailed UI/UX specification. The source of truth for frontend developers |
| [`screenshots/`](./screenshots/) | Screenshots of the interactive prototype |

## Interactive Prototype

The Vercel/v0 prototype is a **high-fidelity interactive UI/UX reference prototype**. It shows how the portal should look and behave.

- **Prototype URL:** `https://v0.app/adhalakhadhanesh-1826/chat/kiss-vocational-management-portal-gpHIk5jPcee`

> The prototype is **not** the production frontend and **not** the final React implementation.

## Documentation

- Start with [`UI_UX_SPEC.md`](./UI_UX_SPEC.md) for the design system, navigation, components, states and role workflows.
- Use [`screenshots/`](./screenshots/) for a quick visual check of the prototype.
- Where the prototype and the specification disagree, **the specification wins**.

## Prototype vs Production Frontend

```
Vercel Prototype = interactive UI/UX reference
React Frontend   = actual implementation
```

| | Vercel / v0 Prototype | React Frontend |
|---|---|---|
| Role | UI/UX reference | Production implementation |
| Location | Vercel deployment (URL above) | `/frontend` |
| Code reuse | Not a code base to copy or extend | Written independently |
| Source of truth | No. Reference only | Follows `UI_UX_SPEC.md` |

## Implementation Status

| Area | Status |
|---|---|
| UI/UX specification (V1) | Frozen |
| Interactive prototype (Vercel/v0) | Reference available. Update URL above |
| Screenshots | Update as screenshots are added |
| React frontend (`/frontend`) | Update as implementation progresses |

## Frontend Development Guidance

- Implement the production UI independently in `/frontend`, following [`UI_UX_SPEC.md`](./UI_UX_SPEC.md).
- Use the prototype and screenshots only to confirm intended look and behaviour.
- Do not copy prototype code into `/frontend`.
- Keep the established terminology and navigation structure from the specification.
- Anything marked **Not specified** in the specification needs a team decision before it is built.
- V2 items in the specification are out of scope for V1.
