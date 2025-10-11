---
toc: false
---

<section class="container">
<div class="hero">
  <h1>TES</h1>
  <h2>Temperature–Emissivity Separation</h2>
</div>


<div class="lead">
    <strong>TES</strong> is the inverse problem of disentangling an object temperature from its spectral emissivity using measured thermal radiance, where they appear multiplicatively as
    (${tex`\varepsilon(\lambda)`}, ${tex`B_\lambda(T)`}). Because ${tex`N`}-band measurements contain ${tex`N`} emissivity values plus one temperature (${tex`N+1`} unknowns).

</div>


```js
const timeline = [
  {year: new Date(1980, 0, 1), label: "ADE", path: "/algorithms/ade", orientation: "up"},
  {year: new Date(1982, 0, 1), label: "CLASSIFICATION", path: "/algorithms/classification", orientation: "down"},
  {year: new Date(2003, 0, 1), label: "ARTEMISS", path: "/algorithms/artemiss", orientation: "up"},
  {year: new Date(2010, 0, 1), label: "ISTA", path: "/algorithms/ista", orientation: "down"}
];
```

```js
import * as d3 from "npm:d3";

function renderTimeline(width) {
  const height = 240;
  const margin = {top: 12, right: 30, bottom: 36, left: 30};
  const innerWidth = Math.max(0, width - margin.left - margin.right);
  const innerHeight = Math.max(120, height - margin.top - margin.bottom);
  const baselineY = innerHeight / 2;

  const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .attr("aria-label", "Algorithms Timeline");

  const g = svg.append("g").attr("transform", `translate(${margin.left},${margin.top})`);

  const x = d3.scaleTime()
    .domain([new Date(1980, 0, 1), new Date(2025, 0, 1)])
    .range([0, innerWidth]);

  // baseline
  g.append("line")
    .attr("x1", 0)
    .attr("x2", innerWidth)
    .attr("y1", baselineY)
    .attr("y2", baselineY)
    .attr("stroke", "#1b1e25");

  // ticks centered on baseline
  const tickEvery = d3.timeYear.every(5);
  const axis = d3.axisBottom(x)
    .ticks(tickEvery)
    .tickFormat(d3.timeFormat("%Y"))
    .tickSize(0);

  g.append("g")
    .attr("transform", `translate(0,${baselineY})`)
    .call(axis)
    .call(g => g.selectAll("text").attr("dy", "1.25em"))
    .call(g => g.selectAll(".tick").append("line")
      .attr("y1", -4).attr("y2", 4).attr("stroke", "#1b1e25"));

  const stem = Math.min(80, innerHeight * 0.36);

  const nodes = g.append("g").selectAll("a")
    .data(timeline)
    .join("a")
      .attr("href", d => d.path)
      .attr("target", "_self");

  // stems (clickable)
  nodes.append("line")
    .attr("x1", d => x(d.year))
    .attr("x2", d => x(d.year))
    .attr("y1", baselineY)
    .attr("y2", d => d.orientation === "up" ? baselineY - stem : baselineY + stem)
    .attr("stroke", "#1b1e25")
    .attr("stroke-width", 2)
    .attr("pointer-events", "stroke");

  // dots
  nodes.append("circle")
    .attr("cx", d => x(d.year))
    .attr("cy", d => d.orientation === "up" ? baselineY - stem : baselineY + stem)
    .attr("r", 3)
    .attr("fill", "#1b1e25");

  // labels
  nodes.append("text")
    .attr("x", d => x(d.year))
    .attr("y", d => d.orientation === "up" ? baselineY - stem - 10 : baselineY + stem + 22)
    .attr("text-anchor", "middle")
    .attr("font-weight", 700)
    .attr("font-size", 14)
    .text(d => d.label);

  return svg.node();
}
```

<div class="timeline">${ resize((width) => renderTimeline(width)) }</div>

</section>



<style>

.container {
  max-width: 980px;
  margin: 0 auto;
  padding: 0 1rem;
}

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 2.5rem 0 3rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 1rem 0;
  padding: 1rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

.lead {
  max-width: none !important;
  margin: 0 auto;
  text-align: center;
  color: var(--theme-foreground-muted);
  font-size: 1rem;
}

.timeline {
  max-width: 980px;
  margin: 1.5rem auto 0;
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

</style>
