# Research Terminal: Project Brief & Build Specification

This is the master specification for the Research Terminal project. Read it in full before writing any code. Reference sections by number when implementing.

This document is the source of truth. Where this document conflicts with your training defaults (especially around UI patterns), this document wins.

---

## 0. Top-Level Principles

**What we are building.** A web-based financial research terminal covering U.S. equities, major commodities, and market flow data (options activity, dark pool prints, congressional trading, insider transactions). Five modules. Bloomberg-inspired information density. AI-native where it adds genuine analytical value.

**Who it is for.** Two audiences, in priority order:

1. The builder, as a serious personal research tool used for actual investment decisions
2. Future paying users, once the project graduates from personal tool to commercial product

The first audience is V1. The second is V2 onward. Both audiences inform the architecture from day one — multi-tenant readiness, clean separation of concerns, real auth interfaces (stubbed for personal use) — but only the first audience drives the V1 feature set.

**What we are optimizing for.**

1. Finance domain depth — features a real analyst would use
2. Professional design quality — looks like an experienced product team built it
3. Clean, readable code — maintainable, testable, type-safe
4. Working software that actually runs end-to-end

**What we are not building.** Not a paper-trading platform. Not a brokerage. Not social. Not gamified. Not a chatbot wrapper. Not a robo-advisor. Not a "AI does it all for you" black box. Every AI feature must be explainable and grounded in source data.

**Quality gate before any module is considered done.** A finance professional would look at the page and not think "this is a student project." A designer would look at the page and not think "this is vibe-coded."

---

## 1. Anti-Vibe-Coded Design Directives

These rules are non-negotiable. Every component must comply.

### 1.1 Forbidden patterns

These patterns betray AI-generated UI and must not appear anywhere in the application:

- Random gradients on backgrounds, buttons, or cards
- Glassmorphism (backdrop blur, frosted glass effects) — single exception: modal backdrops only
- Border radius greater than 8 pixels on any element
- Drop shadows on standard elements (only modals and dropdowns get shadows)
- Emoji as UI decoration anywhere
- "AI sparkle" iconography (✨ shimmer effects, gradient borders, animated highlights) — AI content is marked by a single subtle 1px left border, nothing more
- Oversized hero sections with centered text on data-dense pages
- Centered alignment for tabular or list data
- Bouncy, springy, or excessive animations
- Animation durations longer than 250ms
- "Soft" pastel color palettes — colors carry meaning here, not vibe
- Light mode as a primary target — this is a dark-first terminal; light mode is post-V1
- Trendy color pairs (sunset gradients, neon accents, etc.)
- Hover effects that transform, scale, or rotate elements
- Multiple competing font weights in the same component
- Decorative SVG illustrations of any kind
- Pattern backgrounds, noise textures, or any decorative imagery
- Auto-playing animations or loops
- Skeleton screens that "shimmer" — they are static placeholders only

### 1.2 Required patterns

Every page must follow these conventions:

- **Information density.** This is a terminal, not a marketing site. Default to dense layouts. Whitespace serves grouping, not "breathing room."
- **Monospace numbers.** All numerical data uses JetBrains Mono. Right-aligned in tables. Tabular-nums OpenType feature enabled.
- **Right-aligned numbers, left-aligned labels.** No exceptions in data tables.
- **Color carries meaning.** Red and green appear only for financial deltas (positive/negative changes). Blue is the only accent color. Cyan appears extremely rarely, only for distinguishing a second data series in a chart.
- **Borders, not shadows.** Visual separation uses 1px borders in the border-default color, not box shadows.
- **Subtle hover states.** Hover changes background color one step (bg-elevated → bg-hover). Never changes size, position, or shape.
- **Skeleton loading states.** Every async UI shows skeleton placeholders matching the final layout. No spinners except inline progress on buttons.
- **Empty states.** Every list, table, and panel has a designed empty state with one line of explanatory text and (where appropriate) a single action button.
- **Keyboard-first navigation.** Every primary action has a keyboard shortcut. Cmd+K opens the universal command palette. Arrow keys navigate lists. Esc closes overlays.
- **Consistent component scale.** Buttons, inputs, and table rows all use the 32px standard height. Dense table rows use 28px. Section headers use 40px. No arbitrary sizing.

### 1.3 Reference apps for the look

When in doubt about visual decisions, model the patterns after these production financial and developer tools, in this order:

1. Linear (typography, spacing, command palette, keyboard navigation)
2. Bloomberg Terminal (information density, color discipline, multi-pane layouts)
3. Koyfin (financial dashboard patterns, chart presentation)
4. TradingView (chart UI, technical analysis tooling)
5. Stripe Dashboard (clean financial data tables, no decoration)
6. Vercel Dashboard (header chrome, settings layouts)

Do not model after: Notion templates, generic SaaS landing pages, any "AI-first" trendy app, or anything labeled "modern dashboard UI kit" on Dribbble.

---

## 2. Design System

### 2.1 Color tokens

Configure in `tailwind.config.js`. These are the only colors used in the app. If a color is not in this list, it does not appear in the UI.

```js
colors: {
  bg: {
    base:     '#0B0E14',
    elevated: '#14181F',
    hover:    '#1C212A',
    active:   '#242A35',
  },
  border: {
    subtle:  '#1F242E',
    default: '#2A3340',
    strong:  '#3A4555',
  },
  text: {
    primary:   '#E6EBF5',
    secondary: '#8B95A8',
    tertiary:  '#5A6478',
    disabled:  '#3A4555',
  },
  accent: {
    DEFAULT: '#3B82F6',     // primary blue, used sparingly
    hover:   '#2563EB',
    muted:   '#1E3A8A',
    cyan:    '#06B6D4',     // secondary data series only
  },
  pos: '#10B981',           // positive deltas only
  neg: '#EF4444',           // negative deltas only
  warn: '#F59E0B',          // warning states only
}
```

### 2.2 Typography

Two font families, no others.

```
UI:      'Inter', system-ui, -apple-system, sans-serif
         Weights: 400, 500, 600, 700
Numbers: 'JetBrains Mono', 'Fira Code', monospace
         Weights: 400, 500
         OpenType features: 'tnum' (tabular numbers) always on
```

Type scale. Every text element in the app must use one of these:

| Token | Size / Line Height | Weight | Use |
|-------|---------------------|--------|-----|
| `text-xs` | 11px / 16px | 500 | Table headers, KPI labels (uppercase, letter-spacing 0.4px) |
| `text-sm` | 12px / 16px | 400 | Table cells, secondary metadata |
| `text-base` | 13px / 20px | 400 | Body text, default UI |
| `text-md` | 14px / 20px | 400 | Form inputs, buttons |
| `text-lg` | 16px / 24px | 500 | Section headers, card titles |
| `text-xl` | 20px / 28px | 600 | Page headers |
| `text-2xl` | 28px / 36px | 700 | Large display numbers (price tickers, KPI hero values) |

### 2.3 Spacing scale

Use only these values. No arbitrary spacing.

```
0, 2, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64
```

### 2.4 Border radius

```
0     — sharp edges, used for table cells, dense lists
4px   — inputs, small buttons, tags
6px   — cards, panels, larger buttons
8px   — modals, dropdowns, large containers
```

Anything larger than 8px is forbidden.

### 2.5 Shadows

Almost all separation uses 1px borders, not shadows. Shadows appear only on overlays:

```
modal:    0 4px 16px rgba(0, 0, 0, 0.4)
dropdown: 0 2px 8px rgba(0, 0, 0, 0.3)
popover:  0 2px 8px rgba(0, 0, 0, 0.3)
```

### 2.6 Animation

```
duration-fast:    100ms (hover transitions)
duration-default: 150ms (most transitions)
duration-slow:    200ms (panel transitions, sidebar collapse)
easing:           ease-out
```

No spring physics. No bounce. No stagger.

### 2.7 Component specifications

Build these as reusable components in `frontend/src/components/ui/`. Every page composes from these.

**KPITile** (`KPITile.tsx`)
- Container: `bg-elevated` background, 1px `border-default`, 6px radius, 16px padding
- Label: `text-xs` uppercase, `text-tertiary`, letter-spacing 0.4px, margin-bottom 8px
- Value: `text-2xl` JetBrains Mono, `text-primary`, weight 600
- Delta: `text-sm` JetBrains Mono, color by sign (`pos` / `neg`), with chevron-up/down icon, margin-top 4px
- Optional sparkline: 32px height, full width, 12px below delta
- Min width 140px, no max

**DataTable** (`DataTable.tsx`)
- Header row: 32px height, `bg-elevated`, 1px `border-default` bottom, `text-xs` uppercase, `text-tertiary`, letter-spacing 0.4px
- Body row: 32px height (28px dense mode), no background, 1px `border-subtle` bottom
- Hover row: `bg-hover` background, 100ms transition
- Cells: 8px horizontal padding, no vertical padding (rely on row height)
- Numeric cells: right-aligned, JetBrains Mono, `text-sm`
- Text cells: left-aligned, Inter, `text-sm`
- Sortable columns: chevron-up-down icon at 12px next to header text, very subtle, `text-tertiary`, active sort fills the relevant chevron `text-secondary`
- No zebra stripes
- No row dividers other than 1px border-subtle

**Card** (`Card.tsx`)
- `bg-elevated` background, 1px `border-default`, 6px radius
- Header (optional): 13px medium weight, `text-secondary`, 1px `border-subtle` bottom, 12px padding
- Body: 16px padding standard, 24px for primary cards
- No shadow

**Button** (`Button.tsx`)
- Heights: 28px (sm), 32px (default), 36px (lg)
- Padding: 0 12px default
- Border-radius: 4px
- `text-md` weight 500
- Variants:
  - `primary`: `bg-accent` background, white text, `hover:bg-accent-hover`
  - `secondary`: `bg-elevated` background, 1px `border-default`, `text-primary`, `hover:bg-hover`
  - `ghost`: transparent background, `text-secondary`, `hover:bg-hover hover:text-primary`
  - `danger`: `bg-neg` background, white text
- Disabled state: 40% opacity, no hover
- Loading state: spinner replaces label, button width preserved

**Input** (`Input.tsx`)
- Height 32px, 12px horizontal padding
- `bg-base` background, 1px `border-default`, 4px radius
- `text-md`, `text-primary`
- Placeholder: `text-tertiary`
- Focus: `border-accent`, no glow, no shadow
- Disabled: 50% opacity, `bg-elevated` background
- Optional leading/trailing icon: 16px, `text-tertiary`

**Select** (`Select.tsx`)
- Same chrome as Input. Native select element styled, or Radix Select primitive
- Dropdown panel: `bg-elevated`, 1px `border-default`, 6px radius, dropdown shadow
- Items: 32px height, 12px padding, `text-md`, hover `bg-hover`, selected has checkmark icon

**Tabs** (`Tabs.tsx`)
- Horizontal tab bar: 40px height, 1px `border-default` bottom
- Tab item: 12px horizontal padding, `text-md`, `text-secondary`, 2px transparent bottom border
- Active tab: `text-primary`, 2px `accent` bottom border
- Hover inactive: `text-primary`
- No background fills on tabs, no rounded corners
- Tab badge (optional, for counts): small pill, `bg-elevated`, `text-tertiary`, 10px text, 4px radius

**Badge** (`Badge.tsx`)
- Inline pill, 18px height, 6px horizontal padding, 4px radius
- `text-xs` weight 500
- Variants: neutral (`bg-elevated`, `text-secondary`), positive (`bg-pos/15`, `text-pos`), negative (`bg-neg/15`, `text-neg`), warn (`bg-warn/15`, `text-warn`), accent (`bg-accent/15`, `text-accent`)

**AIInsightCard** (`AIInsightCard.tsx`)
- Same as Card but with 2px left border in `accent` color
- Small "AI" badge in header, `bg-accent/15 text-accent`, no sparkle icon
- The only visual cue that content is AI-generated
- Always disclose the source data the AI synthesized from (e.g., "Based on 12 news articles, last 7 days")

**Toolbar** (`Toolbar.tsx`)
- 48px height, 1px `border-default` bottom
- 16px horizontal padding
- Used at top of every page below TopBar

**Sidebar** (`Sidebar.tsx`)
- Width: 200px expanded, 56px collapsed
- `bg-elevated` background, 1px `border-default` right
- Logo/wordmark area: 48px height, 16px padding, 1px `border-subtle` bottom
- Nav item: 36px height, 12px horizontal padding, `text-md`, `text-secondary`
- Active item: 2px `accent` left border (no fill), `text-primary`
- Hover: `bg-hover`, `text-primary`
- Collapse toggle at bottom

**TopBar** (`TopBar.tsx`)
- 48px height, 1px `border-default` bottom, `bg-base`
- Universal command palette trigger centered: 480px wide, 32px height, `bg-elevated`, 1px `border-default`, 4px radius, placeholder "Search tickers, commands... ⌘K"
- Right side: settings icon, user avatar (stubbed for personal use, becomes real for multi-user)

**CommandPalette** (`CommandPalette.tsx`)
- Triggered by Cmd+K from anywhere
- Modal-style overlay, top of screen, 640px wide, centered horizontally
- Sections: Recent, Tickers (live search), Commands (navigate to module, settings)
- Each result: 36px height, 12px padding, ticker symbol in JetBrains Mono on left, company name in Inter, instrument type badge on right
- Arrow keys navigate, Enter selects, Esc closes

**TickerPreview** (`TickerPreview.tsx`)
- Popover triggered by hovering any ticker symbol anywhere in the app
- 320px wide, `bg-elevated`, 1px `border-default`, 6px radius, popover shadow
- Content: symbol, name, current price, day change, mini 1-month sparkline, key 2 KPIs
- 300ms hover delay before opening
- Click to navigate to full research view

### 2.8 Layout patterns

**Page structure:**

```
┌───────────────────────────────────────────────────────┐
│  TopBar (48px)                                        │
├──────┬────────────────────────────────────────────────┤
│      │  Toolbar (48px) - page title, primary actions  │
│ Side ├────────────────────────────────────────────────┤
│ bar  │                                                │
│      │  Main content area                             │
│ 200px│  (max-width 1600px, centered on wide screens)  │
│      │  24px horizontal padding                       │
│      │                                                │
└──────┴────────────────────────────────────────────────┘
```

**Common content patterns:**

- **Two-column workspace**: left column 320px (filters, inputs), right column flexible (results, visualization). Use for Module 3 structuring, Module 5 scanners.
- **Tabbed detail page**: tab bar below toolbar, content panel below. Use for Module 1 ticker detail.
- **Grid of cards**: 3-4 columns on desktop, 2 on tablet, 1 on narrow. 16px gap. Use for portfolio holdings, KPI grids, idea feed.
- **Master-detail**: left list (320-400px), right detail (flex). Use for Module 5 flow scanners.

---

## 3. Tech Stack

**Backend**

- Python 3.11+
- FastAPI (async)
- Pydantic v2 for all models
- SQLAlchemy 2.0 + Alembic for data persistence
- SQLite for development, PostgreSQL-compatible schemas for future production
- httpx for HTTP requests (async)
- pandas + numpy for analytics
- yfinance for equity and commodity market data
- py_vollib for options pricing and Greeks
- anthropic SDK for AI features
- praw for Reddit
- cachetools for in-memory caching
- pytest + httpx for tests
- python-jose + passlib for auth (interfaces only in V1, full implementation in V2)

**Frontend**

- React 18 with TypeScript (strict mode)
- Vite for build tooling
- Tailwind CSS for styling
- TanStack Query (React Query) for all server state
- Zustand for client state (no Redux, no Context-as-state)
- TradingView lightweight-charts for price charts
- Recharts for analytical charts (correlation heatmaps, payoff diagrams, sector rotation)
- Radix UI primitives for accessible interactive components (Dialog, Select, Tabs, Popover, DropdownMenu, Tooltip)
- lucide-react for icons (no other icon libraries)
- cmdk for command palette
- React Router v6 for routing
- date-fns for date formatting (no moment.js)
- zod for runtime validation

**Infrastructure**

- SQLite for local dev (file at `backend/data/terminal.db`)
- Schema designed PostgreSQL-compatible for V2
- In-memory cache via cachetools TTLCache
- Environment variables via python-dotenv
- V1 deployment target: Railway (backend) + Vercel (frontend)
- V2 commercial target: same infrastructure, scaled up

**Forbidden dependencies**

Do not add any of these without explicit reason:

- Redux, MobX, Recoil — Zustand is sufficient
- styled-components, emotion, vanilla-extract — Tailwind only
- Material UI, Ant Design, Chakra, Mantine, shadcn — build custom on Radix primitives
- Bootstrap, Bulma, any utility CSS framework other than Tailwind
- GraphQL, Apollo, urql — REST is fine
- Socket.io, Pusher — polling is fine for V1
- Docker compose, Kubernetes config — overkill for V1
- Storybook, Chromatic — not needed
- ESLint extends from `airbnb` or similar opinionated configs — minimal config
- Sass, Less — Tailwind only

---

## 4. Directory Structure

```
research-terminal/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                       # FastAPI app entry
│   │   ├── config.py                     # Pydantic Settings
│   │   ├── deps.py                       # Shared dependencies (DB session, current user)
│   │   ├── exceptions.py                 # Custom exception classes
│   │   ├── cache.py                      # TTL cache instance + @cached decorator
│   │   ├── db/
│   │   │   ├── __init__.py
│   │   │   ├── session.py                # SQLAlchemy session factory
│   │   │   ├── base.py                   # Declarative base
│   │   │   └── models.py                 # SQLAlchemy ORM models
│   │   ├── auth/                         # V1: stub; V2: full implementation
│   │   │   ├── __init__.py
│   │   │   ├── current_user.py           # FastAPI dependency, returns user
│   │   │   ├── models.py                 # User Pydantic models
│   │   │   └── stub.py                   # Personal-mode: returns default user
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   ├── research.py               # Module 1
│   │   │   ├── portfolio.py              # Module 2
│   │   │   ├── structuring.py            # Module 3
│   │   │   ├── ideas.py                  # Module 4
│   │   │   ├── flow.py                   # Module 5
│   │   │   └── health.py                 # Health check
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── data/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── yfinance_client.py
│   │   │   │   ├── fred_client.py
│   │   │   │   ├── eia_client.py
│   │   │   │   ├── usda_client.py
│   │   │   │   ├── cftc_client.py
│   │   │   │   ├── sec_client.py
│   │   │   │   ├── finra_client.py       # Module 5: dark pool data
│   │   │   │   ├── congress_client.py    # Module 5: STOCK Act
│   │   │   │   ├── reddit_client.py
│   │   │   │   └── news_client.py
│   │   │   ├── analytics/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── risk.py
│   │   │   │   ├── correlation.py
│   │   │   │   ├── options.py
│   │   │   │   ├── sizing.py
│   │   │   │   ├── options_flow.py       # Module 5: unusual activity detection
│   │   │   │   └── insider_signals.py    # Module 5: cluster buys, etc.
│   │   │   └── ai/
│   │   │       ├── __init__.py
│   │   │       ├── claude_client.py
│   │   │       ├── prompts/              # System prompts as .md files
│   │   │       │   ├── news_summary.md
│   │   │       │   ├── sentiment.md
│   │   │       │   ├── supply_chain.md
│   │   │       │   ├── suggestions.md
│   │   │       │   └── flow_commentary.md
│   │   │       ├── news_summary.py
│   │   │       ├── sentiment.py
│   │   │       ├── supply_chain.py
│   │   │       ├── suggestions.py
│   │   │       └── flow_commentary.py
│   │   └── models/                       # Pydantic schemas (request/response)
│   │       ├── __init__.py
│   │       ├── research.py
│   │       ├── portfolio.py
│   │       ├── structuring.py
│   │       ├── ideas.py
│   │       └── flow.py
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py
│   │   └── (test files mirroring app/ structure)
│   ├── alembic/                          # DB migrations
│   ├── data/                             # SQLite DB lives here (gitignored)
│   ├── requirements.txt
│   ├── pyproject.toml                    # mypy, pytest, ruff config
│   ├── .env.example
│   └── README.md
├── frontend/
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── api/
│   │   │   ├── client.ts                 # axios/fetch wrapper
│   │   │   ├── research.ts               # Module 1 hooks
│   │   │   ├── portfolio.ts              # Module 2 hooks
│   │   │   ├── structuring.ts            # Module 3 hooks
│   │   │   ├── ideas.ts                  # Module 4 hooks
│   │   │   └── flow.ts                   # Module 5 hooks
│   │   ├── components/
│   │   │   ├── ui/                       # Design system primitives (Section 2.7)
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── DataTable.tsx
│   │   │   │   ├── KPITile.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Select.tsx
│   │   │   │   ├── Tabs.tsx
│   │   │   │   ├── Badge.tsx
│   │   │   │   ├── AIInsightCard.tsx
│   │   │   │   ├── Toolbar.tsx
│   │   │   │   ├── ChartCard.tsx
│   │   │   │   ├── EmptyState.tsx
│   │   │   │   ├── ErrorState.tsx
│   │   │   │   └── Skeleton.tsx
│   │   │   ├── layout/
│   │   │   │   ├── AppShell.tsx
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   ├── TopBar.tsx
│   │   │   │   ├── CommandPalette.tsx
│   │   │   │   └── TickerPreview.tsx
│   │   │   ├── research/                 # Module 1 components
│   │   │   ├── portfolio/                # Module 2 components
│   │   │   ├── structuring/              # Module 3 components
│   │   │   ├── ideas/                    # Module 4 components
│   │   │   └── flow/                     # Module 5 components
│   │   ├── pages/
│   │   │   ├── DashboardPage.tsx         # Home / default landing
│   │   │   ├── ResearchPage.tsx
│   │   │   ├── PortfolioPage.tsx
│   │   │   ├── StructuringPage.tsx
│   │   │   ├── IdeasPage.tsx
│   │   │   ├── FlowPage.tsx
│   │   │   └── SettingsPage.tsx
│   │   ├── lib/
│   │   │   ├── formatters.ts             # formatPrice, formatChange, formatCompact, formatDate
│   │   │   ├── cn.ts                     # clsx + tailwind-merge helper
│   │   │   └── constants.ts              # tracked funds, commodity symbols, etc.
│   │   ├── stores/
│   │   │   ├── ui.ts                     # Sidebar state, command palette state
│   │   │   └── recent.ts                 # Recently viewed tickers
│   │   ├── types/
│   │   │   └── index.ts                  # Shared TS types matching backend Pydantic
│   │   ├── hooks/
│   │   │   ├── useKeyboardShortcut.ts
│   │   │   ├── useDebouncedValue.ts
│   │   │   └── useLocalStorage.ts
│   │   └── styles/
│   │       └── globals.css               # Font imports, base Tailwind directives
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   ├── tsconfig.app.json
│   ├── tsconfig.node.json
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── vite.config.ts
│   ├── .env.example
│   └── README.md
├── .gitignore
├── README.md
└── CLAUDE.md                             # This file
```

---

## 5. Authentication & Multi-Tenant Architecture

V1 is a personal-use tool. V2 onward is a paid service. The architecture supports both from day one without overbuilding V1.

### 5.1 Personal mode (V1, current build)

- A single `User` row exists in the database with id 1, email "owner@local"
- `get_current_user` dependency returns this user for every request
- No login UI, no token validation, no session management
- Every model and query that takes a `user_id` uses this default user
- The user avatar in the TopBar links to a stub Settings page

### 5.2 Multi-tenant mode (V2, future)

Designed but not implemented in V1. The following interfaces and conventions must exist from V1:

- `User` SQLAlchemy model exists with id, email, hashed_password, created_at, subscription_tier ("free", "pro", "pro_plus"), api_credits_remaining
- Every business model has a `user_id` foreign key (Portfolio, Watchlist, etc.)
- Every list endpoint filters by `user_id` automatically via the `get_current_user` dependency
- API key handling lives in two places: env vars for shared keys (Finnhub, FRED), per-user fields for personal keys (Anthropic API key for V2 BYOK option)
- Rate limiting middleware is stubbed in V1, real in V2

### 5.3 Subscription tier interface

V2 tiers (these are placeholder, not implemented in V1, but the field exists):

- Free: Module 1 only, 10 ticker views per day, no AI features
- Pro ($29/mo): Modules 1-4, 100 ticker views per day, AI features enabled
- Pro Plus ($99/mo): All modules including Module 5, unlimited views, full AI

`User.subscription_tier` enum exists; gating logic is stubbed (always returns "pro_plus" in V1).

### 5.4 What V1 does NOT include

- Sign-up or login flows
- Password reset
- Email verification
- Stripe integration
- Real subscription enforcement
- Multi-device session management
- Audit logs

These are all V2. Build the data model and interfaces so that adding them later is straightforward.

---

## 6. Shared Infrastructure

### 6.1 Configuration

`backend/app/config.py` loads from `.env`:

```python
class Settings(BaseSettings):
    # AI
    ANTHROPIC_API_KEY: str
    ANTHROPIC_MODEL: str = "claude-sonnet-4-6"
    
    # Data sources
    FRED_API_KEY: str
    EIA_API_KEY: str
    USDA_API_KEY: str
    FINNHUB_API_KEY: str
    REDDIT_CLIENT_ID: str
    REDDIT_CLIENT_SECRET: str
    REDDIT_USER_AGENT: str
    SEC_USER_AGENT: str  # "Name email@example.com" — required by SEC
    
    # Infrastructure
    DATABASE_URL: str = "sqlite:///./data/terminal.db"
    CACHE_TTL_DEFAULT: int = 300
    ALLOWED_ORIGINS: str = "http://localhost:5174,http://localhost:5173"
    
    # Server
    BACKEND_PORT: int = 8001
    LOG_LEVEL: str = "INFO"
    ENV: str = "development"  # development | production
    
    class Config:
        env_file = ".env"
```

Generate `.env.example` with all keys, placeholder values, and a comment for each indicating where to get the key.

### 6.2 Cache layer

`backend/app/cache.py`:

- Module-level `TTLCache` instances by category (prices, fundamentals, news, filings, ai, default)
- TTLs:
  - prices: 60 seconds
  - fundamentals: 24 hours
  - news: 5 minutes
  - filings: 24 hours
  - ai: 1 hour
  - flow data: 5 minutes
  - default: from settings
- `@cached(category="prices")` decorator wraps async functions, uses function name + args hash as key
- Cache stats endpoint for debugging (`GET /api/health/cache`)

### 6.3 Data clients

All data clients inherit from `BaseDataClient`:

```python
class BaseDataClient:
    def __init__(self, api_key: str | None = None):
        self.api_key = api_key
        self.client = httpx.AsyncClient(timeout=10.0)
    
    async def _request(self, method: str, url: str, **kwargs) -> dict:
        # Includes: timeout, retry-on-5xx (max 2), error wrapping to DataSourceError
        ...
```

Each client wraps its specific API and exposes typed methods returning Pydantic models. All public methods use `@cached`.

### 6.4 AI service

`backend/app/services/ai/claude_client.py`:

```python
class ClaudeService:
    def __init__(self, api_key: str, model: str):
        self.client = AsyncAnthropic(api_key=api_key)
        self.model = model
    
    async def complete(
        self,
        system: str,
        user: str,
        max_tokens: int = 1024,
    ) -> str: ...
    
    async def complete_json(
        self,
        system: str,
        user: str,
        schema: type[BaseModel],
        max_tokens: int = 2048,
    ) -> BaseModel: ...
```

`complete_json` instructs the model to return JSON conforming to the schema, parses, validates. On parse failure, retries once with corrective prompt, then raises `AIError`.

System prompts live as Markdown files in `app/services/ai/prompts/` and are loaded at startup.

### 6.5 Error handling

Custom exceptions (`app/exceptions.py`):

- `DataSourceError(message, source, original_error)` → 502 Bad Gateway
- `TickerNotFoundError(symbol)` → 404 Not Found
- `RateLimitError(source, retry_after)` → 429 Too Many Requests
- `AIError(message, original_error)` → 503 Service Unavailable
- `AuthenticationError(message)` → 401 Unauthorized
- `InsufficientCreditsError(remaining, required)` → 402 Payment Required (V2 placeholder, never raised in V1)

FastAPI exception handler returns:

```json
{ "error": { "code": "TICKER_NOT_FOUND", "message": "...", "details": {...} } }
```

### 6.6 Database

SQLAlchemy 2.0 with the following V1 tables:

- `users` (id, email, hashed_password, subscription_tier, api_credits_remaining, created_at)
- `portfolios` (id, user_id, name, risk_profile, created_at, updated_at)
- `positions` (id, portfolio_id, symbol, quantity, cost_basis, purchase_date, created_at)
- `watchlists` (id, user_id, name, created_at)
- `watchlist_items` (id, watchlist_id, symbol, added_at)
- `recent_views` (id, user_id, symbol, viewed_at) — for "Recent" in command palette
- `ai_runs` (id, user_id, feature, prompt_hash, tokens_in, tokens_out, created_at) — usage tracking

Alembic migrations from day one.

### 6.7 Frontend infrastructure

- `frontend/src/api/client.ts`: configured fetch wrapper with base URL from env, JSON parsing, error envelope handling
- All API calls go through TanStack Query hooks in `src/api/*.ts`
- Global QueryClient configured with sensible defaults: staleTime 60s for most queries, refetchOnWindowFocus false
- ErrorBoundary at the page level, generic 500 page for unhandled errors
- Sentry-ready (interfaces only, not implemented V1)

---

## 7. Module 1: Research Tool

**Goal**: Search any U.S. equity or major commodity futures symbol. Get a comprehensive single-page deep dive with price, fundamentals, news, sentiment, holders, supply chain, events.

### 7.1 Supported instruments

**Equities**: any ticker on yfinance (no filter)

**Commodities** (yfinance futures symbols):

| Category | Symbols |
|----------|---------|
| Energy | CL=F (WTI crude), BZ=F (Brent), NG=F (natgas), HO=F (heating oil), RB=F (gasoline) |
| Metals | GC=F (gold), SI=F (silver), HG=F (copper), PL=F (platinum), PA=F (palladium) |
| Grains | ZC=F (corn), ZS=F (soybeans), ZW=F (wheat), ZO=F (oats), ZR=F (rice) |
| Softs | KC=F (coffee), SB=F (sugar), CT=F (cotton), CC=F (cocoa), OJ=F (orange juice) |
| Livestock | LE=F (live cattle), HE=F (lean hogs) |

Auto-detect instrument type from the `=F` suffix.

### 7.2 Backend endpoints

```
GET  /api/research/search?q={query}                        # autocomplete
GET  /api/research/{symbol}                                # aggregated summary
GET  /api/research/{symbol}/price-history?period=...       # 1d|5d|1mo|3mo|6mo|1y|5y|max
GET  /api/research/{symbol}/news?limit=20
GET  /api/research/{symbol}/news-summary                   # AI-summarized
GET  /api/research/{symbol}/sentiment
GET  /api/research/{symbol}/fundamentals                   # equities only
GET  /api/research/{symbol}/commodity-fundamentals         # commodities only
GET  /api/research/{symbol}/holders                        # equities only
GET  /api/research/{symbol}/supply-chain                   # equities only
GET  /api/research/{symbol}/events
GET  /api/research/{symbol}/word-associations
POST /api/research/{symbol}/watchlist                      # add/remove from watchlist
```

### 7.3 Data sources by section

**Price + chart**: yfinance `Ticker.history()`. 60s cache for intraday, 24h for periods ≥ 1mo.

**Fundamentals (equities)**: yfinance `Ticker.info`, `quarterly_financials`, `quarterly_balance_sheet`, `quarterly_cashflow`. Surface market cap, P/E, P/B, EV/EBITDA, profit margins, revenue growth (YoY), FCF, debt/equity, ROE, ROIC, dividend yield, payout ratio.

**News**: Finnhub `/company-news` for equities, general news with keyword filter for commodities. Limit 20.

**News summary (AI)**: Top 10 articles → Claude → structured `NewsSummary`:
- 3-5 bullets "what happened" (factual, neutral)
- 1-2 sentence "why it matters" (analytical)
- 3 tagged themes (e.g., "regulatory", "earnings", "macro")

**Sentiment**: 
- Reddit via PRAW: 50 recent posts from relevant subreddits, Claude scores each pos/neg/neutral with confidence
- News headlines: Claude scores
- Aggregate to -100 / +100 gauge with confidence interval

**Events (equities)**: yfinance earnings dates, ex-div dates, splits. Plus macro events from a sector-event mapping (rate-sensitive sectors get FOMC, broad market gets CPI/NFP).

**Events (commodities)**: hardcoded schedule (EIA Wed for petroleum, Thu for natgas, USDA WASDE monthly, OPEC meetings, Fed meetings).

**Holders (equities)**: SEC EDGAR 13F filings. Use submissions JSON to find recent 13F-HR for the position, parse holdings tables. Top 10 holders + notable QoQ changes.

**Supply chain (equities, AI-native flagship feature)**: Pull most recent 10-K from SEC EDGAR. Extract Business Description and Risk Factors sections. Send to Claude with structured extraction prompt. Return:
- Major customers (with revenue concentration % if disclosed)
- Major suppliers (with concentration if disclosed)
- Key partnerships
- Geographic revenue breakdown
- Regulatory exposure
- Key raw materials / commodities dependence

This is the differentiator. Put extra effort into the prompt engineering.

**Word associations**: Last 30 days of news headlines + top 50 Reddit titles → Claude prompt: "Return 15 most distinctively associated words/phrases for {ticker}, with frequency and sentiment". Display as tag cloud, color-coded.

**Commodity fundamentals**:
- Energy: EIA weekly petroleum status (crude inventories, gasoline, distillates), natural gas storage, 5-year range chart
- Grains/Softs: USDA NASS production estimates, stocks, exports, WASDE highlights
- Metals: COMEX inventories (where available), tracker ETF (GLD, SLV) holdings as proxy
- All: COT positioning (commercial vs managed money net, with percentile context)

### 7.4 Frontend (`/research/:symbol`)

Layout:

```
[Toolbar: symbol input (defaults to current), watchlist toggle, share button]
[Header section: 
  Left: Symbol (text-2xl JBM), full name, instrument type badge
  Right: current price (text-2xl JBM), day change (colored), session indicator (Pre / Open / Post / Closed)
]
[Sparkline strip: 1-day intraday chart, 80px tall, full width]
[Tab bar: Overview | News | Sentiment | Fundamentals | Holders | Supply Chain | Events]

Overview tab content:
  [Two-column row:
    Left (60%): Large price chart with period selector (1D|5D|1M|3M|6M|1Y|5Y|MAX), 480px tall
    Right (40%): KPI tile grid (2x3 or 3x2) — Market Cap, 52W Range, Volume, P/E, Dividend, Beta
  ]
  [AIInsightCard: News summary]
  [Two-column row:
    Left (50%): Sentiment gauge + word cloud preview
    Right (50%): Upcoming events list (next 5)
  ]
```

Tab content for News, Fundamentals, etc. follows similar density-first patterns.

### 7.5 Acceptance criteria

- Search "AAPL" returns full research view, all sections loaded in < 4s on cold cache
- Search "CL=F" returns commodity view with EIA inventories and COT positioning
- Search invalid ticker returns clean error state, not 500
- All number formatting matches conventions (Section 2.2 typography, Section 2.7 components)
- Loading states never show spinners, only skeletons matching final layout
- Supply chain extraction returns useful structured output for AAPL, NVDA, TSLA, XOM, KO
- Page is keyboard-navigable: arrow keys cycle tabs, /  focuses search, Esc closes overlays

---

## 8. Module 2: Portfolio Manager

**Goal**: Upload or enter a portfolio. Get personalized risk analytics and AI-driven suggestions calibrated to a stated risk profile.

### 8.1 Input

- CSV upload: `symbol, quantity, cost_basis, purchase_date` (last two optional)
- Manual entry form
- Multiple portfolios per user (named portfolios)
- Persisted to SQLite (`portfolios` + `positions` tables)

### 8.2 Risk profile

- Conservative / Balanced / Aggressive
- Optional: age bracket, time horizon (years), primary goal (Income / Growth / Preservation)

### 8.3 Backend endpoints

```
GET    /api/portfolio                             # list portfolios for current user
POST   /api/portfolio                             # create portfolio
GET    /api/portfolio/{id}                        # single portfolio
PATCH  /api/portfolio/{id}                        # update profile/name
DELETE /api/portfolio/{id}
POST   /api/portfolio/{id}/upload                 # CSV upload
POST   /api/portfolio/{id}/positions              # add position
PATCH  /api/portfolio/{id}/positions/{pos_id}     # edit
DELETE /api/portfolio/{id}/positions/{pos_id}
GET    /api/portfolio/{id}/analytics              # risk metrics
GET    /api/portfolio/{id}/correlation            # NxN correlation
GET    /api/portfolio/{id}/exposures              # sector/asset class/geo
POST   /api/portfolio/{id}/suggestions            # AI-driven complements
```

### 8.4 Analytics

Computed over 1Y daily data:

- Annualized return, volatility (stdev × √252), beta vs SPY, Sharpe (rf from FRED DGS3MO), max drawdown
- VAR (95%, parametric) and CVAR
- Portfolio beta (weighted), portfolio vol
- Correlation matrix
- Sector exposure (equities via yfinance), commodity exposure (grouped), geographic exposure

### 8.5 AI suggestions

Prompt structure (Section 6.4 `complete_json`):

```
System: You are a portfolio analyst at a long-only fund...
User: Holdings: [positions]. Analytics: [metrics]. Profile: {profile}. Goal: {goal}.
      Identify 3-5 specific gaps. For each, suggest 1-2 specific instruments with thesis,
      suggested weight, and risk consideration. Return JSON conforming to schema.
```

### 8.6 Frontend (`/portfolio`)

Layout:

- Left sidebar: portfolio list (allow switching between user's portfolios)
- Main area:
  - Toolbar: portfolio selector, "New", "Upload CSV", risk profile selector
  - KPI strip: Total Value, Day P&L, Total Return, Sharpe, Max DD, Beta
  - Tabs: Positions | Risk | Exposures | Suggestions
  - Positions: editable DataTable with hover-row delete and inline-edit
  - Risk: KPI grid + correlation heatmap (Recharts) + drawdown chart
  - Exposures: stacked bar (sector + commodity), geographic table
  - Suggestions: card grid, each card with thesis, ticker, weight, "Add" button

### 8.7 Acceptance criteria

- 10-position CSV analyzed in < 5s
- Correlation matrix renders cleanly for 30 positions
- AI suggestions return within 10s, are specific (not generic)
- Risk profile changes meaningfully shift suggestions
- CSV upload validates schema and shows clear errors

---

## 9. Module 3: Investment Structuring

**Goal**: For a candidate trade, help size, hedge, structure, and understand its event risk.

### 9.1 Backend endpoints

```
POST /api/structuring/size
POST /api/structuring/hedge
GET  /api/structuring/{symbol}/options-chain
POST /api/structuring/payoff
GET  /api/structuring/{symbol}/event-risk
```

### 9.2 Position sizing

Methods: fixed fractional, fractional Kelly (0.25-0.5x), volatility-targeted.

Input: account size, ticker, conviction (1-5), stop loss %, sizing method, optional win rate/RR estimates.

Output: dollar amount, share count, risk per trade, % portfolio, brief reasoning.

### 9.3 Hedge suggestions

For equities: sector inverse ETF, SPY short with beta adjustment, correlated peer short.

For commodities: inverse commodity ETF, related FX hedge, equity proxy hedge (XLE for crude, etc.).

Beta-adjusted hedge ratio always computed and shown.

AI rationale for each hedge option (Claude).

### 9.4 Options chain & payoff builder

Options chain via yfinance. For each contract: IV, Greeks (delta/gamma/theta/vega) via py_vollib. Highlight high volume/OI ratio.

Payoff builder: structure-by-leg input, computed payoff curve, breakevens, max P/L, capital required.

Preset structures: long call, long put, covered call, protective put, vertical spreads (bull/bear), straddle, strangle, iron condor, iron butterfly.

### 9.5 Event risk

For a position, surface all events in next 90 days: earnings, ex-div, FOMC, CPI/NFP, sector-specific (EIA for energy, WASDE for ags), OPEC meetings.

Color-coded by historical impact (heuristic).

### 9.6 Frontend (`/structuring`)

Three tabs: Sizing | Hedging | Options.

Two-column workspace per tab: inputs left (320px), results right (flex).

Payoff diagram: Recharts area chart with line at zero, breakevens marked, max profit/loss labeled. Draggable strike sliders.

### 9.7 Acceptance criteria

- Sizing produces sensible numbers across all 3 methods (cross-check against position sizing calculators)
- Options chain renders for AAPL, TSLA, SPY, NVDA
- Payoff diagrams correct for all preset structures (cross-check against CBOE)
- Greeks match reference calculator within 0.5% tolerance

---

## 10. Module 4: Investment Ideas

**Goal**: Surface non-obvious ideas from public filings and positioning data.

### 10.1 Backend endpoints

```
GET /api/ideas/13f-flows?lookback_days=90
GET /api/ideas/funds                                # list of tracked funds
GET /api/ideas/funds/{cik}                          # specific fund detail
GET /api/ideas/cot-extremes
GET /api/ideas/sector-rotation
GET /api/ideas/feed                                 # aggregated cross-source feed
```

### 10.2 13F flow scanner

Track ~20 notable funds (hardcoded CIKs): Berkshire, Tiger Global, Bridgewater, Citadel, Renaissance, Pershing Square, Third Point, Greenlight, Appaloosa, Baupost, Coatue, Lone Pine, Viking, Maverick, Two Sigma, AQR, D.E. Shaw, Millennium, Point72, Soros.

Per filing (45-day delayed):
- Compare to prior quarter
- New positions (> $5M notional)
- Increased positions (> 20%)
- Eliminated positions
- Top 10 holdings

Display: filterable DataTable + per-fund detail page.

### 10.3 COT extremes

Weekly COT data via CFTC Socrata (`publicreporting.cftc.gov/resource/6dca-aqww.json`).

For each tracked commodity:
- Managed money net position as % of open interest
- 1Y, 3Y, 5Y percentile
- Flag top 5% / bottom 5%
- Heatmap: commodities × time, color = percentile

AI commentary on flagged extremes (Claude): what it means, recent analogs.

### 10.4 Sector rotation

Relative strength of sector ETFs (XLK, XLF, XLE, XLV, XLI, XLY, XLP, XLU, XLB, XLRE, XLC) vs SPY over 1M, 3M, 6M.

Display: RRG-style scatter (X = momentum, Y = relative strength) or ranked bar chart fallback.

### 10.5 Idea feed

Aggregated chronological feed pulling from all sources. Each item is a typed event ("13F: Citadel added $200M NVDA", "COT: managed money short crude at 5Y extreme", "Sector: XLE breaking down vs SPY"). Click → detail page.

### 10.6 Acceptance criteria

- 13F scanner works for all 20 tracked funds
- COT correctly identifies known extremes
- Feed has 20+ items per week in normal market

---

## 11. Module 5: Market Flow & Signals (Unusual Whales Equivalent)

**Goal**: Surface non-public-equity signal in near-real-time. Options flow, dark pool prints, congressional trading, insider transactions, ETF flows. This is the flagship module for personal trading use.

### 11.1 What this replaces

The data Unusual Whales charges $48/mo for, sourced from publicly available filings and feeds. Not as fast or polished as their product. Comparable analytical value for the categories below.

### 11.2 Backend endpoints

```
# Options flow
GET /api/flow/options/{symbol}                         # flow for one ticker
GET /api/flow/options/unusual                          # market-wide unusual activity scanner
GET /api/flow/options/flow-leaders?period=1d|5d|30d    # tickers with highest premium flow

# Dark pool
GET /api/flow/dark-pool/{symbol}                       # dark pool activity per ticker
GET /api/flow/dark-pool/recent                         # recent large prints market-wide

# Congressional
GET /api/flow/congress                                 # recent disclosures
GET /api/flow/congress/by-ticker/{symbol}
GET /api/flow/congress/by-member/{member_id}
GET /api/flow/congress/members                         # list of tracked members

# Insiders
GET /api/flow/insiders/{symbol}                        # Form 4 transactions for ticker
GET /api/flow/insiders/recent                          # market-wide recent insider buys/sells
GET /api/flow/insiders/cluster-buys                    # multi-insider clusters

# ETFs
GET /api/flow/etf-flows                                # major ETF inflows/outflows

# Scanner
POST /api/flow/scanner                                 # configurable cross-source scanner
```

### 11.3 Data sources by section

**Options flow** (yfinance + computed metrics):

Pull full options chain via yfinance. For each contract, compute:
- Volume / OI ratio (unusual if > 2)
- Premium (price × volume × 100) (large if > $500k)
- Premium relative to past 30-day average for that strike (anomaly if > 3σ)
- Net implied direction (call premium - put premium)

Mark a chain entry "unusual" if it crosses any threshold. The unusual scanner returns these across all tracked tickers (start with S&P 500 + popular meme tickers, ~600 symbols), refreshed every 5 minutes.

Limitations: yfinance is delayed, so this is not real-time. Useful for daily-and-larger flow analysis. Disclose this clearly in the UI.

**Dark pool data** (FINRA):

FINRA publishes daily ATS (Alternative Trading System) data with a 2-week delay for individual ATS, and short sale daily volume on T+1.

Sources:
- FINRA OTC Transparency Data: `https://otctransparency.finra.org/otctransparency/` (downloadable CSVs)
- FINRA Short Sale Volume Daily Files: free download by date

Surface:
- ATS volume as % of total daily volume per ticker
- Large prints (where derivable from daily summaries)
- Short sale ratio changes

Disclose the T+1 / 2-week lag clearly. This is structural for free data; paid Unusual Whales uses different feeds.

**Congressional trading** (Senate EFD + House clerk):

The Senate Office of Public Records publishes financial disclosures at `https://efdsearch.senate.gov`. House disclosures at `https://disclosures-clerk.house.gov`. Both are scrapeable but inconsistent in format.

Approach:
- Scrape recent filings on a daily cron (or on-demand if cron infrastructure is V2)
- Parse for transaction date, ticker, transaction type (Buy/Sell), amount range, member
- Persist to SQLite
- Display chronologically and by ticker

Alternative: there is an unofficial scraping library `congress-trades` on PyPI that handles the parsing. Use it if it works, fall back to direct scraping if it breaks.

**Insider transactions** (SEC EDGAR Form 4):

SEC Form 4 filings are filed within 2 business days of a transaction. EDGAR full-text search supports filtering by form type.

Endpoint: `https://efts.sec.gov/LATEST/search-index?q=&forms=4&dateRange=custom&startdt=...&enddt=...`

Parse:
- Transaction date, type (open market buy/sell, exercise, gift, etc.)
- Insider name, role
- Shares, price, total value
- Resulting position size

Compute "cluster buy" signal: multiple insiders (3+) of the same company buying within 14 days. Strong historical signal.

Filter out exercises and grants (only count open-market purchases for the strongest signal).

**ETF flows**:

Daily ETF flow data is paywalled at most providers. Free alternatives:

1. Compute approximate flows from daily change in shares outstanding (ETF prospectuses publish this via NSCC files)
2. Use ETF.com or VettaFi free pages (scrape)
3. Use yfinance for total return and AUM proxy

V1: implement basic version showing 1-day, 5-day, 30-day flow estimates for major ETFs (SPY, QQQ, IWM, sector ETFs, themed ETFs). Disclose methodology.

**Scanner**:

Configurable cross-source scanner. User builds a query like:

- Tickers with: (Unusual options activity AND insider cluster buy in last 14 days)
- Or: (Congressional buy in last 30 days AND positive sentiment shift)

Frontend is a query builder. Backend translates to a database query + cache lookup.

### 11.4 Analytics

`backend/app/services/analytics/options_flow.py`:

- `compute_unusual_score(contract, historical_volumes)` → float 0-100
- `detect_sweeps(chain)` → list of likely sweep orders (large size across multiple strikes)
- `aggregate_directional_flow(chain)` → net bullish/bearish premium

`backend/app/services/analytics/insider_signals.py`:

- `detect_cluster_buys(transactions, window_days=14)` → list of cluster events
- `score_insider_transaction(txn)` → score weighted by insider role, transaction size, voluntary vs scheduled

### 11.5 AI commentary

`backend/app/services/ai/flow_commentary.py`:

For each significant flow event, generate a 1-sentence contextual note: "Cluster buy at NVDA — 4 insiders purchased $2.3M in past 10 days, CEO is the largest buyer". Use sparingly to avoid LLM cost runaway.

### 11.6 Frontend (`/flow`)

Toolbar: page title "Market Flow", filter chips (timeframe, sectors), scanner button.

Tab bar: Options | Dark Pool | Congress | Insiders | Scanner.

**Options tab**:
- Top: scanner table (DataTable) — symbol, type (call/put), strike, expiry, vol/OI ratio, premium, unusualness score
- Sortable by every column
- Click row → drills into ticker's full options chain
- Right side: filter panel (min premium, min vol/OI ratio, expiry range, calls only / puts only / both)

**Dark Pool tab**:
- DataTable of recent large prints
- Columns: symbol, price, size, dollar value, time, % of daily volume, ATS

**Congress tab**:
- DataTable of recent disclosures
- Columns: filed date, member, party, ticker, transaction type, amount range, days from txn to disclosure
- Filter by party, chamber, ticker, member
- Per-member detail page accessible by clicking name

**Insiders tab**:
- Two sections: "Recent Transactions" (DataTable) and "Cluster Buys" (Card grid, each card a ticker with cluster event details)
- Filter: insider role, transaction type, dollar range

**Scanner tab**:
- Query builder UI: pick conditions from dropdowns, AND/OR logic
- Saved scans persist per user
- Results show as DataTable with same drill-down behavior

### 11.7 Acceptance criteria

- Options scanner returns 20+ flagged contracts on a normal trading day across S&P 500
- Congress data refreshed daily, shows latest disclosures within 24-48h of filing
- Insider cluster detection identifies known clusters (validate against published examples)
- Dark pool data shows for at least 50 highest-volume tickers daily
- Scanner can express "find me tickers where Pelosi bought in last 30 days AND IV rank > 50" or similar combined query

---

## 12. Cross-Module Features

### 12.1 Command palette (Cmd+K)

Triggered from anywhere. Sections in order:

1. Recent: last 5 viewed tickers (from `recent_views` table)
2. Tickers: live autocomplete (debounced 200ms) against ticker search endpoint
3. Watchlist: user's watchlist tickers
4. Commands: "Go to Portfolio", "Go to Flow > Options", "Open Settings", etc.

Keyboard: ↑↓ navigate, Enter selects, Esc closes, Tab cycles sections.

### 12.2 Ticker preview popover

Every ticker symbol rendered anywhere in the app is wrapped in a `<TickerPreview symbol="..." />` component. 300ms hover delay → popover opens with mini summary. Click → navigate to research page.

### 12.3 Keyboard shortcuts

- `Cmd+K` — command palette
- `/` — focus current page's primary search
- `g r` — go to Research (vim-style sequential bindings)
- `g p` — go to Portfolio
- `g s` — go to Structuring
- `g i` — go to Ideas
- `g f` — go to Flow
- `?` — show shortcut help overlay
- `Esc` — close any overlay

### 12.4 Settings page (`/settings`)

Stub for V1. Sections:
- Profile (read-only stub email)
- API Keys (BYOK display, V2 only)
- Theme (dark only in V1, light coming)
- Keyboard shortcuts reference
- About / version

### 12.5 Dashboard / home (`/`)

Default landing page. Layout:

- Top: watchlist as horizontal scrollable ticker tape with current prices
- Two-column grid:
  - Left: News summary AIInsightCard ("What's moving today" — aggregated)
  - Right: Recent activity (cluster buys, COT extremes, options flow leaders)
- Below: 4-card grid linking to each module with a short description and a relevant stat

---

## 13. Quality Bar

### 13.1 Backend

- Full type hints; `mypy --strict` passes on `app/`
- All endpoints have Pydantic request and response models
- Pytest coverage ≥ 60% on `services/` modules
- No `print()`; structured `logging` only
- External API calls: 10s timeout, retry-on-5xx (max 2), wrap errors in `DataSourceError`
- All AI prompts in versioned `.md` files
- No secrets in code or version control

### 13.2 Frontend

- TypeScript `strict: true`, `noUncheckedIndexedAccess: true`
- No `any` without a `// reason` comment
- All API calls go through TanStack Query hooks
- Every async UI has loading skeleton + error state
- No console errors in production build
- `npm run typecheck` passes
- `npm run lint` passes (minimal ESLint config: TS recommended + react-hooks)
- All keyboard shortcuts work
- All interactive elements are accessible (proper aria, focus rings via `outline-none focus-visible:ring-2 focus-visible:ring-accent`)

### 13.3 Design

- Every component conforms to Section 2 spec
- No forbidden patterns (Section 1.1) anywhere
- Numbers in JetBrains Mono, right-aligned, formatted with sign for deltas
- Dark theme only; light theme is V2
- Page renders correctly at 1280, 1440, 1920 widths
- Mobile: pages render usably at 768; below that, show "Best viewed on desktop" banner (V1 acceptable)

### 13.4 Git hygiene

- Conventional commits (`feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `docs:`)
- One logical change per commit
- Commit messages describe the *what* and *why*, not the *how*
- Branch name format: `feat/module-1-research`, `fix/options-chain-empty-state`
- PRs (when working solo, still useful): one PR per module sub-feature

---

## 14. Build Plan: Single-Shot Execution

This build plan is designed for autonomous execution by Claude Code in a single session. Section 16 contains the master build prompt.

The build proceeds in 8 phases. Each phase must be committed before moving to the next. If context window pressure becomes an issue mid-build, Claude Code should commit current progress, write a `BUILD_STATE.md` file noting exactly where it stopped and what is next, and end the session for resumption.

### Phase A: Foundation (≈ scaffolding + design system)
- Scaffold both projects (backend FastAPI, frontend Vite React TS)
- Install all dependencies from Section 3
- Tailwind config per Section 2 (colors, typography, spacing)
- Font setup (Inter, JetBrains Mono via Google Fonts or self-hosted)
- Base CSS with reset and tabular-nums utility
- Build all UI primitive components from Section 2.7
- Set up routing, app shell, sidebar, top bar, command palette, ticker preview popover
- Set up TanStack Query, Zustand, ErrorBoundary
- Commit: `feat: foundation (scaffolding, design system, app shell)`

### Phase B: Backend infrastructure
- Config, cache, error handling, exception handler
- ClaudeService (async, complete + complete_json)
- BaseDataClient
- SQLAlchemy setup, models, Alembic init, first migration
- Auth stub (default user, get_current_user dependency)
- Health check endpoints (basic + cache stats)
- Pytest setup with conftest, mocks for external APIs
- Tests: cache decorator, ClaudeService, exception handler, base data client
- Commit: `feat: backend infrastructure (db, cache, ai service, auth stub)`

### Phase C: Module 1 — Research Tool
- yfinance, FRED, EIA, USDA, CFTC, SEC, Reddit, news clients
- AI services: news_summary, sentiment, supply_chain
- All Module 1 endpoints (Section 7.2)
- Frontend: research page with all 7 tabs, all data wired
- Commit: `feat(module-1): research tool — equities and commodities`

### Phase D: Module 5 — Market Flow (built before Modules 2-4 because of personal-use priority)
- FINRA, Congress, Form 4 clients
- Options flow analytics (unusual scoring, sweep detection)
- Insider signals analytics (cluster buys)
- All Module 5 endpoints (Section 11.2)
- Frontend: flow page with 5 tabs
- Commit: `feat(module-5): market flow and signals`

### Phase E: Module 4 — Investment Ideas
- 13F scanner (using SEC client from Module 1)
- COT extremes (CFTC client)
- Sector rotation analytics
- Idea feed aggregator
- All Module 4 endpoints
- Frontend: ideas page
- Commit: `feat(module-4): investment ideas`

### Phase F: Module 2 — Portfolio Manager
- Portfolio CRUD + CSV upload
- Risk analytics (risk.py, correlation.py)
- AI suggestions (uses ClaudeService.complete_json)
- All Module 2 endpoints
- Frontend: portfolio page
- Commit: `feat(module-2): portfolio manager`

### Phase G: Module 3 — Investment Structuring
- Sizing (sizing.py)
- Hedging logic
- Options chain + Greeks (options.py with py_vollib)
- Payoff diagram builder
- Event risk endpoint (reuses event sources from Modules 1, 4)
- Frontend: structuring page
- Commit: `feat(module-3): investment structuring`

### Phase H: Polish
- Dashboard/home page (Section 12.5)
- Settings page stub
- Watchlist functionality
- Recent views tracking
- Empty states and error states across the app
- README with screenshots and architecture diagram
- One-paragraph "Notable features" section in README highlighting Module 5 and supply chain extraction
- Commit: `chore: polish, dashboard, watchlists, docs`

### Realistic expectations

Claude Code in one session may not complete all 8 phases. The phase order prioritizes the most distinctive features (research tool + market flow) for personal use, so partial completion still yields a usable product. If the session ends mid-phase, the build state file should make resumption straightforward.

---

## 15. Deployment & Future Commercial Path

### 15.1 V1 deployment

- Backend: Railway, hobby tier ($5/mo)
- Frontend: Vercel, free tier
- Domain: optional (use the default *.vercel.app for V1)
- Env vars set in respective dashboards
- CORS: production frontend URL added to `ALLOWED_ORIGINS`

### 15.2 V2 commercial readiness

When transitioning to a paid service:

1. Add proper auth (Supabase Auth or Auth0 — both have good free tiers)
2. Add Stripe subscription billing tied to `users.subscription_tier`
3. Implement real rate limiting per tier (FastAPI middleware → Redis counter)
4. Migrate SQLite → managed PostgreSQL (Railway Postgres, Supabase, or Neon)
5. Add usage tracking (`ai_runs` table already exists for this)
6. Add a landing page (separate from the app)
7. Add legal pages (Terms, Privacy)
8. Disclaimer: "Not investment advice" on all AI features and suggestions
9. Audit log table for compliance posture

None of this is V1. The architecture supports all of it.

### 15.3 Compliance considerations

When V2 is real:

- "Not investment advice" disclaimer is required throughout
- AI suggestions cannot be presented as recommendations from a person — must be clearly AI-generated
- If targeting U.S. users, do not present yourself as a Registered Investment Advisor unless registered
- Data licensing: check ToS for each data source on commercial use (most free sources allow commercial use but some require attribution or restrict redistribution)

---

## 16. Master Build Prompt

This is the prompt to paste into Claude Code after `CLAUDE.md` is in place.

> Read CLAUDE.md in full before doing anything else. Then execute the entire build per Section 14, autonomously, without asking for permission between steps. Auto-approve all file edits, command runs, and dependency installs. Treat my pre-existing approval as standing.
>
> Build order is strict: Phase A → B → C → D → E → F → G → H.
>
> After each phase: run any tests you wrote for that phase, fix any failures, then commit with the exact message from Section 14. Do not move to the next phase until tests pass and commit succeeds.
>
> Section 1 (anti-vibe-coded directives) and Section 2 (design system) are not suggestions. Every UI component must conform. Reject any default UI pattern that conflicts with these sections in favor of the spec's pattern.
>
> Section 5 multi-tenant architecture: implement the interfaces (User model, get_current_user dependency returning default user, user_id foreign keys everywhere) but stub authentication. Do not implement login UI, password handling, or session management.
>
> Module 5 is critical and must be implemented in full per Section 11. The data sources are public — scrape or pull via APIs as specified. Honestly disclose latency limitations in the UI (e.g., "yfinance options data is delayed", "FINRA dark pool data is T+1").
>
> Use the current best Claude Sonnet model for AI features. Read the model name from settings (ANTHROPIC_MODEL env var).
>
> If you hit context window pressure: stop at a phase boundary, commit current work, write `BUILD_STATE.md` to the repo root noting (1) which phase completed last, (2) which phase is next, (3) any partial work in progress with file paths, (4) any known issues to resume on. Then end the session cleanly so I can resume in a fresh session.
>
> If anything in CLAUDE.md is ambiguous, make the more conservative design choice (less decoration, more density, more standard pattern) and note the decision in your commit message.
>
> Begin Phase A now.

---

End of brief.
