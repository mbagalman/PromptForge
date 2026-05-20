# A Practitioner's Guide to Dashboard Design

*An 80/20 reference for designing BI dashboards in Tableau, Power BI, Looker, or any similar tool. Tool-agnostic — focused on what to put on the screen and why, not how to drag fields onto a canvas.*

---

## The one idea that matters most

**Start with the decision, not the data.** Dashboards fail when they're built around "what data we have" rather than "what action a user needs to take." Every metric, chart, and pixel on the screen should trace back to a specific decision someone makes. If it doesn't, cut it.

This single principle drives almost every other rule in this guide. The rest is mostly applied common sense.

---

## 1. Define the dashboard before you build it

Before opening Tableau or Power BI, answer these in plain English:

- **Who is the user?** One primary persona per dashboard — executive, manager, analyst, or operator. These people have different jobs, different time horizons, and different tolerance for detail. Trying to serve all of them on one screen produces something that serves none of them.
- **What decisions does this user make from this dashboard, and on what cadence?** Daily operational tweaks? Weekly resourcing calls? Quarterly strategy reviews?
- **What 3 questions must it answer?** A good starting set: *Are we on track? Where are we off track? What should we do about it?* Anchor every element to one of these.
- **What is the "10-second test"?** A user should be able to glance at the dashboard and answer the top question within ten seconds. If they can't, it's too cluttered or the hierarchy is wrong.

A useful framing from business analysis is **Goal → Question → Metric**: state the strategic goal, derive the questions that would tell you whether you're meeting it, then pick metrics that answer those questions. Working in this order keeps vanity metrics off the screen.

One more diagnostic: ask "what should *not* happen?" This surfaces regulatory limits, data-access constraints, and edge cases (missing or wrong data) early, before they become embarrassing post-launch surprises.

---

## 2. Pick the right metrics

A few rules cover most of what matters:

- **5–9 metrics per screen, max.** This is the working-memory ceiling. Aim for 3–5 primary KPIs plus supporting context. More than that and adoption drops.
- **Only include metrics the user can influence.** If they can't act on it, why is it there? "Total page views" without a conversion lens is decoration.
- **Every metric needs context.** A raw number is a fact; with comparison, it's an insight. The minimum useful KPI card has four things: *current value, comparison (vs. target/prior period/benchmark), trend (a sparkline), and variance.*
- **Tier your metrics.** Tier 1 (3–5 primary KPIs, biggest and boldest) → Tier 2 (supporting/explanatory) → Tier 3 (drill-down detail, often a separate page).
- **Mix leading and lagging indicators.** Lagging metrics (revenue, churn) tell you what happened. Leading metrics (pipeline velocity, signups, NPS) tell you what's coming. Both belong, but leading indicators are what enable action.
- **Match metrics to review cadence.** Operational metrics (uptime, queue length) update in real time. Process metrics (handle time, on-time delivery) review weekly. Strategic metrics (retention, NPS) review monthly or quarterly. Don't put a metric on an operational dashboard if it only meaningfully moves once a quarter.
- **Document every metric.** A short data dictionary entry — owner, source, formula, filters, rounding — prevents the slow drift where "revenue" means three different things in three different dashboards.

---

## 3. Choose the right chart

Match chart to question, not to taste. Most analytical questions fall into four buckets (this is Abela's framework, and it holds up):

| Question type | Best charts | Example | Avoid |
|---|---|---|---|
| **Comparison over time** | Line, area (continuous); column (few periods) | Monthly revenue vs. target | Pie; too many lines (>5) |
| **Comparison across categories** | Sorted horizontal bar | Sales by region | Pie — angles are hard to compare |
| **Composition (part-to-whole)** | Stacked bar, treemap | Revenue share by product | Pie with >5 slices |
| **Relationship / correlation** | Scatter, bubble | Marketing spend vs. leads | Dual-axis lines (implies false correlation) |
| **Distribution** | Histogram, box plot | Ticket resolution times | — |
| **Exact value lookup** | Table with conditional formatting | Overdue invoices | Chart requiring eyeball estimation |
| **Single KPI / progress to goal** | KPI card, bullet chart | Quarterly revenue vs. target | Gauges (low precision, too much space) |
| **Geographic patterns** | Choropleth | Sales by state | Maps when geography isn't the point |

A few general rules that catch most mistakes:

- **Bars for comparisons, lines for continuous time.** Don't use a line chart for unordered categories or a bar chart for a smooth time trend.
- **Sort bar charts by value**, not alphabetically, unless the order itself carries meaning.
- **Use tables when users want precise numbers** (operations, finance) and charts when they want patterns. Both have legitimate uses; don't visualize numbers people need to read precisely.
- **Annotate.** A vertical line marking a product launch or campaign explains more than any caption.
- **Limit to ~6–8 visuals per screen.** When you have many series, use small multiples instead of a tangle of overlapping lines.

A quick gut-check: show a chart to a colleague with no context and ask what they see. If they can't explain it in a few seconds, simplify it.

---

## 4. Lay it out so the eye flows

Eyes scan in F- or Z-patterns: top-left first, then across, then down. Design with that, not against it.

The most reliable layout pattern is an **inverted pyramid**:

- **Top — Status:** Large KPI cards answering "are we on track?" Comparison values and variance are non-negotiable here.
- **Middle — Trends and comparisons:** Line and bar charts explaining the *why* behind the top-line numbers.
- **Bottom — Detail:** Tables, drill-downs, owner contacts, links into source systems where action gets taken.

Other guidance worth following:

- **Put the most important thing in the top-left.** Always. This is where users look first.
- **Use whitespace generously** — roughly 30–40% empty space. Density is not depth; it's noise.
- **Align to a grid.** Misaligned panels read as sloppy and increase cognitive load even when users can't articulate why.
- **Group related metrics visually** — proximity, light borders, or shared background tints signal "these belong together."
- **Filters go above the content, with plain labels.** Don't make users hunt.

---

## 5. Use color as signal, not decoration

- **Pick one accent color for "attention"** and use it sparingly. If everything is highlighted, nothing is.
- **Use color consistently across the dashboard.** If green = good in one chart, it should mean the same thing in every chart.
- **Never encode meaning with color alone.** About 8% of men and 0.5% of women have some form of color-vision deficiency. Pair color with an icon, text label, or shape. "Red = bad" plus "▼ -12%" works; red alone doesn't.
- **Prefer blue/yellow over red/green** for status when accessibility matters; red/green is the worst combination for the most common color blindness types.
- **Limit your palette.** Three or four colors with intentional meaning beats a rainbow.

While you're thinking about accessibility: aim for WCAG AA contrast (4.5:1 for body text, 3:1 for large text and chart elements), keep body text at 14–16px minimum, and use a clean sans-serif. These are cheap to get right at design time and expensive to retrofit.

---

## 6. Three dashboard archetypes — pick one

These have different goals and shouldn't be mixed on the same screen:

- **Strategic** — for executives. High-level KPIs, period-over-period comparisons, monthly/quarterly cadence. Few metrics, lots of context.
- **Operational** — for managers and frontline staff. Real-time or near-real-time, alert-driven, action-oriented. "Something's broken — here's what and where."
- **Analytical** — for analysts. Interactive, drill-down-heavy, exploratory. More tolerant of density because users come with specific questions.

If you find yourself trying to make one dashboard serve two archetypes, that's usually a sign it should be two dashboards.

---

## 7. Be deliberate about interactivity

Interactivity is a tool, not a virtue. Every filter, drill-down, or hover adds friction for users who come back daily.

- **Default the dashboard to the most-asked view.** Don't make users configure it every visit.
- **Add filters only where users genuinely have different needs.** A "region" filter is usually justified; a "metric type" filter often hides indecision about what to put on the screen.
- **Use tooltips for definitions and detail**, not for content the user needs at a glance.
- **Drill-down should have a clear path back.** Users get lost when "more detail" navigates them somewhere they can't easily exit.

---

## 8. Treat it as an iterative product, not a deliverable

Dashboards that get used are ones that evolve. A practical iteration loop:

1. **Wireframe before you build.** Sketch the layout on paper or in Figma — boxes labeled "trend chart," "KPI card." Get sign-off on structure before getting tangled in tool specifics.
2. **Prototype with 7–10 metrics.** Resist the urge to ship the complete vision in v1.
3. **Watch real users.** Usage analytics (which widgets get viewed, which get ignored), brief interviews, and the occasional "show me how you used this last week" call surface more than any survey.
4. **Edit, don't rebuild.** Most improvements are subtractions or small refinements. Full redesigns are usually a sign that step 1 was skipped.
5. **Review quarterly.** Business priorities shift; metrics that were essential six months ago may now be noise.

Add two pieces of infrastructure that pay for themselves: an explicit **"Last Updated" timestamp** on every dashboard (silently stale data is the fastest way to destroy trust), and **automated data-quality checks** that flag freshness or completeness issues before users notice them.

---

## A short pre-launch checklist

Before you call a dashboard done:

- [ ] Every metric maps to a specific decision the user makes
- [ ] 5–9 metrics on the primary screen, no more
- [ ] Top-left answers the top question in under 10 seconds
- [ ] Every chart type matches the question it's answering
- [ ] Color is consistent across the dashboard and means the same thing everywhere
- [ ] No information is encoded by color alone
- [ ] Each KPI has a comparison value (target, prior period, or benchmark)
- [ ] The dashboard renders cleanly at the screen size users actually have
- [ ] "Last Updated" timestamp is visible
- [ ] At least one real user has tried it and could answer the top questions without prompting

---

## The shortest possible summary

Design backwards from the decision. Cut anything that doesn't drive action. Give every number context. Match chart to question. Put the most important thing in the top-left. Use color as signal, not decoration. Iterate based on real use.

Almost everything else is detail.
