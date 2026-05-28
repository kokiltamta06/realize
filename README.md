# ₹ealize Finance — Product Design Case Study

> A personal finance dashboard designed for young working professionals in India.
> Built as a zero-dependency, single-file web app — no backend, no login, no friction.


---

## About This Project

₹ealize Finance is a self-initiated project I designed and built end-to-end to solve a real gap — most budgeting tools are either too complex, require cloud accounts, or are built around Western financial patterns that don't reflect how people in India manage money (UPI-first lifestyle, EMI stacking, informal income).

The design constraint was deliberate: the entire product had to live in a single HTML file, work fully offline, and deliver a complete financial picture in under two minutes of input. This constraint shaped every design decision.

This document covers my full design process — research, architecture, component logic, AI-assisted workflows, and technical implementation decisions — as part of my product design portfolio.

---

## My Role

**Solo Product Designer & Front-End Implementer**

| Responsibility | Method |
|---|---|
| User research & problem framing | Qualitative interviews, competitive analysis |
| Information architecture | Flow mapping, card sorting |
| Wireframing & prototyping | Low-fi sketches → Figma → live HTML prototype |
| Visual & interaction design | Custom design system, CSS micro-animations |
| AI-assisted design workflows | Claude (Anthropic) for iteration, code generation, copy |
| Usability review | Heuristic evaluation, think-aloud sessions |

Building the final implementation myself gave me direct control over spacing systems, transition timing, and component state behaviour — eliminating the gap between design intent and delivered output.

---

## AI in My Design Workflow

This project was designed using a **human-led, AI-assisted workflow**. I used [Claude (Anthropic)](https://claude.ai) as a design and development collaborator throughout the process. Below is a transparent breakdown of where and how AI was used — and where it was not.

### Where AI accelerated my process

| Stage | AI Usage | My Contribution |
|---|---|---|
| **Competitive research** | Rapid summarisation of fintech app patterns (YNAB, Walnut, Fi Money) | Framed the right questions, evaluated outputs critically |
| **Copy & microcopy** | Generated label variants for category names, error states, empty states | Selected, edited, and contextualised all final copy |
| **CSS implementation** | Translated design decisions (spacing, transitions, colour tokens) into production CSS | Defined the design system — AI executed the syntax |
| **Component iteration** | Generated multiple toggle, card, and modal layout variants from my specs | Evaluated options against UX criteria, made final calls |
| **PDF export logic** | Built html2canvas + jsPDF pipeline from my specified requirements | Defined the export layout, verified output quality |
| **README & documentation** | Drafted structure and content from my design notes | Reviewed, restructured, and rewrote for accuracy and voice |

### Where AI was not used
- **Design decisions** — layout hierarchy, colour semantics, component behaviour, and interaction patterns were all made by me
- **User research** — interviews, synthesis, and insight generation were done manually
- **Information architecture** — the three-column cognitive flow model was defined by me before any implementation began
- **Design rationale** — all reasoning in this document reflects my own thinking

> **My view on AI in design:** AI handles execution speed. The designer still owns the problem framing, the taste, and the judgment. Using AI without understanding the output is a risk; using it as a force multiplier for a clear-thinking designer is a genuine skill advantage.

---

## Problem Statement

> *How might we help young salaried professionals in India understand and manage their monthly finances — without adding complexity to their lives?*

### Competitive analysis snapshot

| Product | Strength | Gap for Indian users |
|---|---|---|
| YNAB | Powerful envelope budgeting | Subscription cost, US-centric, requires bank sync |
| Walnut | SMS-based auto-tracking | Discontinued, privacy concerns |
| Fi Money | Modern UX, savings focus | Requires account + linked bank |
| Google Sheets | Total flexibility | High friction, no structure for non-finance users |
| **₹ealize** | Zero friction, offline, India-first | — |

### Key pain points from research
- Mandatory account creation creates a trust barrier before the product delivers any value
- Western budget categories (401k, mortgage) create cognitive friction for Indian users
- Bank CSV exports exist but require manual interpretation — no tool bridges this for non-technical users
- Users want scenario modelling ("what if I cut dining?") but don't want to lose their data doing it

---

## Information Architecture

The dashboard maps directly to the sequential mental model users described in research:

```
What do I earn?
      ↓
What do I spend — and on what?
      ↓
What's left over?
      ↓
Am I financially healthy? Am I on track for my goal?
```

This cognitive sequence is expressed spatially as a left → centre → right layout:

```
┌─────────────────────┬──────────────────────────────┬─────────────────────┐
│   LEFT PANEL        │      CENTRE GRID             │   RIGHT PANEL       │
│                     │                              │                     │
│  Monthly Income     │  Needs  │  Wants            │  Projected Savings  │
│  Financial Health   │─────────┼──────────         │  Savings Goal       │
│                     │Emergency│  Debt             │                     │
└─────────────────────┴──────────────────────────────┴─────────────────────┘
```

The centre grid uses a 2×2 layout matching the four-quadrant mental model users naturally described. Left and right panels are fixed-width (280px) with the centre grid consuming remaining space — prioritising the data-entry area.

---

## Key Design Decisions

### 1 — Three-column fixed layout over a tabbed flow
Early explorations used a tabbed interface (Income → Expenses → Summary). In walkthroughs, users consistently tried to see income and expenses simultaneously when thinking about savings. The all-visible layout eliminates navigation entirely — there is only one screen state.

**Technical implication:** Required a `grid-template-columns: 280px 1fr 280px` layout with `overflow: hidden` on the body and independent scroll on the centre panel only.

### 2 — Non-destructive category toggling
Users needed to model scenarios without losing data. The toggle switch disables a category's contribution to the total calculation without removing rows. This required separating the `sectionState` object from the DOM — toggling updates state and triggers `calculate()` rather than hiding or deleting elements.

### 3 — Health score reduced to a single ratio
I evaluated three scoring models:
- **50/30/20 rule compliance** — too prescriptive, fails for debt-heavy users
- **Multi-axis radar chart** — visually complex, hard to act on
- **Savings ratio (savings ÷ income × 100)** — single, actionable, universally understood ✓

The donut chart at `cutout: 85%` is proportioned to read as a gauge rather than a pie — the ring thickness (7.5% of diameter) is intentional. The score overlaid at the centre creates a single focal point.

### 4 — CSV parser with noise-stripping logic
Indian bank statement CSVs contain heavily encoded transaction descriptions (e.g. `UPI-SETTLEMENT-ZOMATO@OKAXIS-REF123456789`). A regex-based cleaning pipeline strips UPI prefixes, reference numbers, and bank codes to surface readable merchant names. Column detection uses fuzzy header matching against known Indian bank export formats (HDFC, ICICI, SBI, Axis, Kotak).

### 5 — Preview-before-export pattern
A direct "Export PDF" button would produce output the user hasn't verified. The preview modal shows a rendered export sheet — styled distinctly from the dashboard (white background, print-optimised layout) — before any file is generated. This follows a standard confirm-before-commit pattern that reduces export errors and builds confidence in the output.

---



## Component Specifications

| Component | Spec | Design Rationale |
|---|---|---|
| **Expense row** | `grid: 1fr 80px 28px`, 8px gap | Label dominates; amount is fixed-width for visual alignment; remove is minimal |
| **Card border radius** | `14px` | Softer than typical SaaS (8px) — approachable for a personal finance context |
| **Toggle switch** | 36×20px, 14px thumb, 200ms ease | Proportioned for thumb-tap on mobile future iteration |
| **Donut chart** | `cutout: 85%`, 140px canvas | Ring width ≈ 10px — readable as a gauge at this size |
| **Progress bar** | 6px height, `border-radius: 10px` | Deliberately thin — goal card is secondary information |
| **Input focus** | `border: 1px solid #c8d0d8`, bg → white | Low-contrast focus ring reduces visual noise in data-entry context |
| **Modal backdrop** | `rgba(0,0,0,0.4)` + `backdrop-filter: blur(4px)` | Blurred background maintains context without full occlusion |

---

## Interaction & Animation Spec

| Interaction | Duration | Easing | Purpose |
|---|---|---|---|
| Splash exit | 800ms | `ease-in-out` | Smooth brand reveal |
| Splash logo entrance | 800ms | `cubic-bezier(0.175, 0.885, 0.32, 1.275)` | Elastic overshoot — memorable |
| Toggle thumb | 200ms | `ease` | Confirms state change without drawing attention |
| Goal progress fill | 300ms | `ease` | Reward feedback on number entry |
| Input focus border | 150ms | `ease` | Fast enough to feel responsive |
| Live recalculation | ~0ms | — | Synchronous JS — no perceived delay |

Live recalculation (savings + health score updating on every `oninput` event) was a deliberate choice over an on-blur or on-submit pattern. In usability walkthroughs, synchronous feedback made users feel in control and encouraged exploratory behaviour — entering higher numbers to see the health score improve before settling on accurate figures.

---

## What I Learned

**On AI-assisted design workflows:**
The most productive use of AI was in the gap between design decision and code output — translating a spacing decision or animation spec into CSS without looking up syntax. This kept me in design thinking mode rather than context-switching to implementation details. The risk is over-delegating judgment; I found it important to always verify AI-generated code against the original design intent before accepting it.

**On constraint-driven design:**
The single-file constraint eliminated scope creep more effectively than any project brief. When every feature competes for space in one file, unnecessary features don't survive. The calculator, CSV parser, and PDF export all earned their inclusion — features that didn't pass that test were cut.

**On designing for two output contexts:**
The export sheet required designing for a static, print-optimised layout within a dynamic, screen-optimised product. This is a non-trivial design problem — what reads well at 1440px wide on a dashboard can fail at 595px wide on an A4 page. Designing the export sheet as a completely separate layout system was the right call.

---

## Accessibility Notes

Current state and known gaps for a future audit:

- ✅ Colour contrast on text meets WCAG AA on white surfaces
- ✅ Interactive elements have visible focus states
- ⚠️ Donut chart has no ARIA label — screen reader users cannot access the data it represents
- ⚠️ Toggle switches use CSS-only — no `role="switch"` or `aria-checked` attributes
- ⚠️ Modal does not trap focus — keyboard users can tab out of the overlay
- ⚠️ No responsive breakpoints — layout breaks below ~1024px

These are documented as known issues and are prioritised in the roadmap.

---

## Roadmap

- [ ] ARIA compliance pass (chart, toggles, modal focus trap)
- [ ] Mobile-responsive layout (target: 375px+)
- [ ] Dark mode (CSS variable swap, no JS required)
- [ ] Monthly history — persistent multi-month comparison
- [ ] Multiple budget profiles
- [ ] AI-powered transaction categorisation on CSV import
- [ ] Onboarding flow for first-time users

---

## Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| Structure & logic | HTML5 / Vanilla JS | No framework — reduces load time to ~0ms |
| Styling | CSS3 custom properties | Design token system via `:root` variables |
| Charting | [Chart.js 4](https://www.chartjs.org/) | Doughnut only — loaded via CDN |
| PDF rendering | [html2canvas](https://html2canvas.hertzen.com/) + [jsPDF](https://github.com/parallax/jsPDF) | Canvas screenshot → A4 PDF with multi-page support |
| Persistence | `localStorage` | Serialised JSON, keyed by version string |
| Typography | Google Fonts (DM Sans + DM Mono) | Loaded async — fallback to system sans-serif |
| AI tooling | [Claude (Anthropic)](https://claude.ai) | Design iteration, code generation, documentation |

---

