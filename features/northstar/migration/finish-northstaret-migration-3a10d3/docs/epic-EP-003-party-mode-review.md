---
doc_type: epic-party-mode-review
epic: EP-003
epic_title: "Shared Chart Primitives (LineGraph + StackedBarGraph)"
phase: devproposal
initiative: finish-northstaret-migration-3a10d3
date: "2026-02-26"
lead: Winston (Architect)
participants:
  - Winston (Architect)
  - Mary (Analyst)
  - Sally (UX Designer)
readiness_verdict: READY
party_verdict: PASS
blocking_issues: 0
---

# Epic Stress Gate: EP-003 — Shared Chart Primitives

## Implementation Readiness Check (adversarial)

**Winston asks:** Is the D3-to-Recharts migration provably complete with no feature regression?

| Check | Question | Answer |
|-------|---------|--------|
| D3 audit complete | Are all custom D3 drawing operations from the Angular source inventoried? | ✅ Yes — tech-decisions.md ADR-004: D3 operations catalogued: `line()`, `area()`, `scaleBand()`, `axisBottom()`, `axisLeft()`, tooltips via `d3-tip`, responsive via `viewBox` |
| Recharts equivalent | Does Recharts cover `d3.area()` path? | ✅ Yes — `<AreaChart>` + `<Area>` with gradient fill |
| Custom tooltip | Is the custom tooltip render prop approach decided? | ✅ Yes — architecture §6.3: `renderCustomTooltip` prop on `<LineGraphChart>` and `<StackedBarChart>` |
| Print mode | Is `isAnimationActive={!isPrintMode()}` the complete print fix? | ✅ Yes — confirmed by existing `isPrintMode()` utility in codebase |
| Responsive container | Who controls `ResponsiveContainer` aspect ratio? | ✅ Yes — parent `<ChartWrapper>` sets `aspect={16/9}`, chart does not hardcode dimensions |
| Zero-data state | Is empty state for charts fully specced? | ✅ Yes — architecture §6.5: `<EmptyChartState>` component shown when `data.length === 0` |
| Color tokens | Are chart colors specification-locked? | ⚠️ Tech-decisions lists recharts default palette — must use NS4 brand tokens (`#0066CC`, `#00A86B`, `#E84545`) |
| Legend accessibility | Is chart legend keyboard-navigable? | ⚠️ Recharts Legend does not expose keyboard nav by default — must add custom accessible legend |

**Deadlines and risks:**
- Color token gap: not a blocker — added to acceptance criterion
- Legend a11y gap: not a blocker — custom `<AccessibleLegend>` story added to EP-003

**Readiness verdict: READY.** Two acceptance-criterion additions only.

---

## Party Mode Review

**Winston (Architect — lead):**
Adversarial challenge #1: `ResponsiveContainer` with `aspect={16/9}` inside a flex layout can cause infinite resize loops in some browsers when the parent container has no fixed height. Mitigation: parent must have `height: min-content` or explicit height. Story acceptance criterion: "Chart container must have a minimum height of 200px set on the parent wrapper." 

Adversarial challenge #2: Recharts uses SVG and D3 used SVG — but our print CSS must target `.recharts-wrapper` not `.d3-chart`. Dev must audit print.css and replace old selectors. Add to story acceptance criteria.

✅ Both are one-liner additions — not blocking.

**Mary (Analyst):**
Chart primitives drive several teacher-facing analytics reports. The `LineGraphChart` must support at least **two independent Y-axes** (e.g., score vs. attendance on the same time axis) — this is used in Section Reports. Recharts supports `<YAxis yAxisId="left" />` and `<YAxis yAxisId="right" orientation="right" />`. The architecture §6.2 mentions dual-axis but does not confirm which stories require it. I want to see "dual-axis support" explicitly in at least one EP-003 story acceptance criterion, not just EP-004. ✅ Not blocking — add to chart primitives data API story.

**Sally (UX Designer):**
Color tokens are critical. NS4 has an established color system. Charts must not ship with Recharts defaults (blue/green/red that clash with NS branding). Story acceptance criterion for color: "All chart series colors use design tokens from `tokens.ts` — no hardcoded hex or Recharts defaults." 

Legend readability: legend items should be 14px/regular weight, with a color swatch (12×12px square, 4px radius). This is a cosmetic detail but sets expectation for visual QA. ✅ Not blocking.

---

## Verdict

| Reviewer | Vote | Notes |
|----------|------|-------|
| Winston | ✅ PASS | Parent height guard + print.css selector audit |
| Mary | ✅ PASS | Dual-axis support must appear in EP-003 chart story |
| Sally | ✅ PASS | Color tokens enforced, legend spec added |

**Result: ✅ READY — no blocking issues. EP-003 clears stress gate.**
