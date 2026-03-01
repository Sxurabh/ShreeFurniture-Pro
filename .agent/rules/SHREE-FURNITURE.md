---
trigger: always_on
---

---
trigger: always_on
priority: P0
---


# SHREE-FURNITURE.md — Project Rules for Antigravity
## Overrides and extensions specific to the Shree Furniture Platform

> **Priority:** This file is P0 alongside GEMINI.md. Rules here are project-specific overrides.
> When a rule here conflicts with GEMINI.md defaults, this file wins.

---

## 🔴 MANDATORY: Session Start Protocol

Before responding to ANY request in this project, you MUST read all of these **in order**:

1. **`STATUS.md`** — what is already built (prevents regenerating existing code)
2. **`PREFERENCES.md`** — owner's design preferences (overrides your UI defaults)
3. **`REJECTIONS.md`** — patterns explicitly rejected by the owner (never suggest these)
4. **`STACK-CHANGES.md`** — if entries exist, they supersede CLAUDE.md tech stack table
5. **`CLAUDE.md`** — hard technical rules (read after checking for stack overrides)
6. **`NewDocs/design-reference.md`** — IKEA-inspired design baseline (**frontend work only** — read before generating any component, page, or UI element)

Then identify the current phase from `STATUS.md` before proposing anything.

**Failure to check `STATUS.md` = regenerating already-built code. Critical failure.**
**Failure to check `REJECTIONS.md` = suggesting something the owner already rejected. Also a failure.**
**Failure to check `design-reference.md` before frontend work = components that don't match the visual language. Also a failure.**

---

## 🤖 Agent Routing — Shree Furniture Specific

### This project uses these agents ONLY:

| Domain | Agent | When |
|---|---|---|
| Next.js storefront, UI, components | `@frontend-specialist` | Any storefront code |
| MedusaJS backend, subscribers, workflows | `@backend-specialist` | Any backend code |
| PostgreSQL schema, migrations | `@database-architect` | Schema or query work |
| CI/CD, Railway, Vercel, Docker | `@devops-engineer` | Infrastructure |
| Vitest unit tests, Playwright E2E | `@test-engineer` | Any test file |
| Cart, pricing, checkout bugs | `@debugger` | Unexpected behavior |
| Core Web Vitals, LCP, ISR | `@performance-optimizer` | Performance tasks |
| Multi-domain feature (e.g., checkout flow) | `@orchestrator` | Spans frontend + backend |

### Agents NOT relevant to this project (do not activate):
- `@game-developer` — not applicable
- `@mobile-developer` — web-only (mobile-responsive, not React Native)
- `@penetration-tester` — use `@security-auditor` instead
- `@seo-specialist` — for Phase 2 only; use `@frontend-specialist` for basic meta tags

---

## 🧩 Skill Loading — Shree Furniture Specific

### Always-Active Skills for This Project

| Skill | Why Always Active |
|---|---|
| `clean-code` | TypeScript strict; no `any`; all exports typed |
| `api-patterns` | All API calls through `lib/medusa/` wrappers |
| `frontend-design` | shadcn/ui + Tailwind v4 + design tokens in `packages/ui` |
| `database-design` | PostgreSQL 16, MikroORM (via Medusa), soft deletes |
| `lint-and-validate` | TypeScript strict, Zod for runtime validation |

### Load on Demand (When Relevant)

| Skill | Load When |
|---|---|
| `testing-patterns` + `webapp-testing` | Writing Vitest or Playwright files |
| `deployment-procedures` | Modifying `.github/workflows/` or Railway/Vercel config |
| `security-auditor` | Webhook handlers, auth flows, env var handling |
| `performance-profiling` | Working on LCP, Core Web Vitals, ISR strategy |
| `seo-fundamentals` | generateMetadata(), JSON-LD schemas, sitemap |

### Skills to NEVER Load (Not Relevant to This Project)

- `game-development` (all sub-skills)
- `mobile-design` (this is a web project, mobile-responsive only)
- `prisma-expert` (we use MikroORM via MedusaJS, not Prisma)
- `nestjs-expert` (we use MedusaJS, not NestJS)
- `python-patterns` (Node.js / TypeScript only)
- `powershell-windows` (Linux/macOS dev environment)

---

## 📐 Project-Specific Code Rules

### Rule 1 — Money is Always Paise (Integers)
```
❌ const price = 49999.00
✅ const price = 4999900  // paise
✅ Display only: formatPrice(4999900) → "₹49,999"
```

### Rule 2 — All Medusa Calls Through `lib/medusa/` Wrappers
```
❌ medusaClient.store.product.list() in a component
✅ import { getProducts } from "@/lib/medusa/products"
```

### Rule 3 — No `any` in TypeScript
```
❌ const data: any = response
✅ const data: unknown = response; if (isProduct(data)) { ... }
```

### Rule 4 — Always Check MedusaJS Version Before Suggesting API
```
This project uses MedusaJS v2.
v1 patterns (medusa.products.list(), medusa.carts.create()) will break.
ALWAYS read NewDocs/MEDUSA-V2-PATTERNS.md before writing backend or client code.
```

### Rule 5 — Rendering Strategy Must Match Architecture
```
PDP / PLP / Homepage → ISR (revalidate: 60)
Search              → CSR (Algolia InstantSearch)
Cart / Checkout     → CSR
Order Confirm       → SSR (no-store fetch)
Account pages       → CSR
```

### Rule 6 — Auth Tokens in HTTP-only Cookies
```
❌ localStorage.setItem("token", jwt)
❌ Zustand store for auth token
✅ Medusa v2 SDK with { auth: { type: "session" } } — cookies handled automatically
```

---

## 📦 Import Conventions (Enforced)

```typescript
// Order: Node built-ins → external → workspace → app aliases → relative
import { createHmac } from "crypto"
import { z } from "zod"
import type { Product } from "@shree/types"
import { formatPrice } from "@/lib/utils/price"
import { ProductCard } from "./ProductCard"
```

---

## 🚫 Anti-Patterns — Auto-Fail Triggers

If you find yourself about to do ANY of the following, **STOP and re-read CLAUDE.md**:

- Building product/order admin UI (→ use Medusa Admin as-is)
- Implementing password hashing or JWT logic (→ MedusaJS handles this)
- Writing a discount/coupon engine (→ Phase 2, out of scope)
- Using Prisma (→ we use MikroORM via Medusa)
- Using localStorage for anything (→ HTTP-only cookies only)
- Adding a `float` for a monetary value (→ paise integers only)
- Building Reviews & Ratings, Loyalty, or AR features (→ Phase 3)
- Calling Medusa API directly from a component (→ lib/medusa/ wrappers only)
- Using MedusaJS v1 SDK methods (→ v2 only, see MEDUSA-V2-PATTERNS.md)

---

## 🎨 Design System Quick Reference

```
Primary:  #1A1A1A   Accent:  #C8A96E   Warm BG: #F5F0E8   Border: #E8E0D0
Heading:  Playfair Display, serif
Body:     Inter, sans-serif
Max W:    1280px    Mobile: 360px min   Breakpoints: sm/md/lg/xl (Tailwind)
```

**Always use `packages/ui/src/tokens.ts` constants. Never hardcode hex values in JSX classNames.**

---

## ✅ Definition of Done (Per Task)

Before marking any task complete, verify:

- [ ] TypeScript compiles: `pnpm type-check` passes with no errors
- [ ] Lint passes: `pnpm lint` passes
- [ ] Relevant unit tests exist and pass (especially for price logic, form validation, webhooks)
- [ ] `STATUS.md` updated to reflect what was completed
- [ ] No `any` types without comment justification
- [ ] No hardcoded secrets or API keys
- [ ] `<Image>` components have explicit `width` and `height`
- [ ] New API calls go through `lib/medusa/` wrappers
- [ ] No new dependencies added without checking if one already in use covers the use case

---

*Shree Furniture | v1.0 — Q1 2026*


================================================
FILE: .agent/scripts/auto_preview.py
================================================
#!/usr/bin/env python3
"""
Auto Preview - Antigravity Kit
==============================
Manages (start/stop/status) the local development server for previewing the application.

Usage:
    python .agent/scripts/auto_preview.py start [port]
    python .agent/scripts/auto_preview.py stop
    python .agent/scripts/auto_preview.py status
"""

import os
import sys
import time
import json
import signal
import argparse
import subprocess
from pathlib import Path

AGENT_DIR = Path(".agent")
PID_FILE = AGENT_DIR / "preview.pid"
LOG_FILE = AGENT_DIR / "preview.log"

def get_project_root():
    return Path(".").resolve()

def is_running(pid):
    try:
        os.kill(pid, 0)
        return True
    except OSError:
        return False

def get_start_command(root):
    pkg_file = root / "package.json"
    if not pkg_file.exists():
        return None
    
    with open(pkg_file, 'r') as f:
        data = json.load(f)
    
    scripts = data.get("scripts", {})
    if "dev" in scripts:
        return ["npm", "run", "dev"]
    elif "start" in scripts:
        return ["npm", "start"]
    return None

def start_server(port=3000):
    if PID_FILE.exists():
        try:
            pid = int(PID_FILE.read_text().strip())
            if is_running(pid):
                print(f"⚠️  Preview already running (PID: {pid})")
                return
        except:
            pass # Invalid PID file

    root = get_project_root()
    cmd = get_start_command(root)
    
    if not cmd:
        print("❌ No 'dev' or 'start' script found in package.json")
        sys.exit(1)
    
    # Add port env var if needed (simple heuristic)
    env = os.environ.copy()
    env["PORT"] = str(port)
    
    print(f"🚀 Starting preview on port {port}...")
    
    with open(LOG_FILE, "w") as log:
        process = subprocess.Popen(
            cmd,
            cwd=str(root),
            stdout=log,
            stderr=log,
            env=env,
            shell=True # Required for npm on windows often, or consistent path handling
        )
    
    PID_FILE.write_text(str(process.pid))
    print(f"✅ Preview started! (PID: {process.pid})")
    print(f"   Logs: {LOG_FILE}")
    print(f"   URL: http://localhost:{port}")

def stop_server():
    if not PID_FILE.exists():
        print("ℹ️  No preview server found.")
        return

    try:
        pid = int(PID_FILE.read_text().strip())
        if is_running(pid):
            # Try gentle kill first
            os.kill(pid, signal.SIGTERM) if sys.platform != 'win32' else subprocess.call(['taskkill', '/F', '/T', '/PID', str(pid)])
            print(f"🛑 Preview stopped (PID: {pid})")
        else:
            print("ℹ️  Process was not running.")
    except Exception as e:
        print(f"❌ Error stopping server: {e}")
    finally:
        if PID_FILE.exists():
            PID_FILE.unlink()

def status_server():
    running = False
    pid = None
    url = "Unknown"
    
    if PID_FILE.exists():
        try:
            pid = int(PID_FILE.read_text().strip())
            if is_running(pid):