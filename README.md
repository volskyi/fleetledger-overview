# FleetLedger: overview

> **Source code is private.** This repository describes the product and its architecture.
> Access for a technical review is available on request.

**A self-hosted ERP for road-transport and fleet operators, built around trip-level profitability.**

![FastAPI](https://img.shields.io/badge/API-FastAPI-009688)
![Next.js](https://img.shields.io/badge/Frontend-Next.js-000000)
![PostgreSQL](https://img.shields.io/badge/DB-PostgreSQL-4169E1)
![PWA](https://img.shields.io/badge/PWA-offline--ready-5A0FC8)
![i18n](https://img.shields.io/badge/i18n-ES%20%2F%20RU-informational)

## The problem

Small and mid-sized carriers run on spreadsheets. Trips, fuel, repairs, leasing and driver pay live in
separate sheets, and the question that matters ("which truck, which customer, which line of business
actually makes money?") gets answered differently every month.

FleetLedger started as a faithful port of a real carrier's Excel model (around 30 vehicles, three business
units). The rule was to reproduce the same business logic exactly before adding anything new, and to test
it against the reference figures from the original workbook.

## What it does

**Operations**
- Trip ledger as the core table: revenue, fuel, tolls, other costs, days, trips, extras
- Vehicle register (type, leasing/financing, odometer, status), drivers, customers, rate cards
- Trip planning with truck-aware distance and toll estimation

**Profitability and payroll**
- Per-trip *direct profit* and *real profit* (after imputed driver salary)
- Per-vehicle P&L with revenue split by business unit, €/km and margin
- Profitability per business unit and per customer
- Driver payroll with selectable pay schemes (fixed, per day, per km, per trip, mixed) and configurable proration
- Period dashboard with fixed-cost proration and trends

**Billing and documents**
- Issued and received invoices (VAT books), delivery notes, receivables aging
- Expiry alerts for documents and maintenance (inspection, insurance, tachograph, service intervals)

**Platform**
- Role-based access (admin, operator, read-only, driver) with 2FA and an audit log
- **AI assistant**, provider-independent: read-only analytics plus a *propose → confirm → execute → revert*
  flow for safe writes; every dialogue is logged
- **Multi-tenant SaaS**: each organisation gets its own database, subdomain and users; a platform operator
  creates, suspends, archives and removes tenants from an admin UI or CLI
- Telematics-ready integration layer (GPS, tachograph hours, CAN-bus fuel, cold-chain temperature)
- Driver-facing PWA, bilingual UI (Spanish / Russian), one-click full data export with no lock-in

## Architecture

```
 Browser / driver PWA
        │
   Next.js (static export)
        │  REST
   FastAPI ── SQLAlchemy / Alembic ── PostgreSQL (one database per tenant)
        │
   AI assistant (pluggable LLM provider, tool calls gated by confirm/revert)
```

- Domain calculations live in one tested module; derived views (payroll, P&L, dashboards) are computed, not stored
- Docker Compose for dev and prod; a demo seed generates realistic data so no client data is ever needed for development
- QA suite checks the computed figures against the reference spreadsheet

## Why not a general ERP?

Odoo-class suites cover accounting, CRM and inventory broadly, but their fleet modules track vehicles and
costs without native per-trip / per-customer profitability. FleetLedger does one industry deeply: small
surface, learnable in a day, cheap to host, with the domain's quirks (driver pay schemes, fuel
double-counting, business-unit splits) built in.

## My role

Product owner and builder: requirements from the operator, domain model, calculation spec, architecture,
implementation, QA against reference data, deployment.
