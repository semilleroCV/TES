---
toc: false
---

<link rel="stylesheet" href="/algorithms/algorithm.css">

<div>
<a href="/" class="alg-back" aria-label="Back to home">←</a>
</div>

<div class="alg-container">
  <header class="alg-hero">
    <h1>Emissivity Bounds</h1>
    <p>Temperature and Emissivity Estimation from Thermal Multispectral Data</p>
  </header>

  <section class="alg-meta">
    <div><strong>Authors</strong>: S. Jaggi, D. Quattrochi, R. Baskin</div>
    <span class="sep"></span>
    <div><strong>Date</strong>: 1992</div>
  </section>

  <section class="alg-section alg-narrow">
    <p>The emissivity bounds method addresses the fundamental challenge in thermal remote sensing: separating temperature from emissivity. With N spectral channels, we have N+1 unknowns (N emissivities and 1 temperature) but only N equations. Rather than assuming constant emissivity or using iterative optimization, this method computes realistic bounds on both parameters.</p>

    <h3 style="margin:2rem 0 1rem">The Underdetermined Problem</h3>
    
    <p>For each channel <em>n</em>, the measured radiance ${tex`F_n`} relates to temperature ${tex`T`} and emissivity ${tex`\varepsilon_n`} through the band-integrated blackbody function:</p>

    <p>${tex.block`F_n = \varepsilon_n B_n(T)`}</p>

    <p>where ${tex`B_n(T)`} is the band-integrated Planck function for channel <em>n</em>.</p>

```js
// Constants for TIMS-like channels
const h = 6.62607015e-34;
const c = 299792458;
const k = 1.380649e-23;

// Simplified TIMS channel centers
const channels = [
  {name: 'Channel 1', lambda: 8.3, a: 0.05, b: 0.003, c: 0.000001},
  {name: 'Channel 2', lambda: 8.65, a: 0.06, b: 0.0035, c: 0.0000012},
  {name: 'Channel 3', lambda: 9.1, a: 0.07, b: 0.004, c: 0.0000014},
  {name: 'Channel 4', lambda: 9.8, a: 0.08, b: 0.0045, c: 0.0000015},
  {name: 'Channel 5', lambda: 10.6, a: 0.09, b: 0.005, c: 0.0000016},
  {name: 'Channel 6', lambda: 11.2, a: 0.10, b: 0.0055, c: 0.0000017}
];
```

```js
// Band-integrated blackbody function (W/cm²/sr)
function B_n(T, channel) {
  return channel.a + channel.b * T + channel.c * T * T;
}
```

```js
// Inverse function: get T from F and epsilon
function getTemp(F, eps, channel) {
  const a = channel.a - F / eps;
  const discriminant = channel.b * channel.b - 4 * channel.c * a;
  if (discriminant < 0) return null;
  return (-channel.b + Math.sqrt(discriminant)) / (2 * channel.c);
}
```

```js
const T_true = 285;
const eps1_true = 0.98;
const eps2_true = 0.97;
const F1 = eps1_true * B_n(T_true, channels[0]);
const F2 = eps2_true * B_n(T_true, channels[1]);

const eps_range = d3.range(0.85, 1.001, 0.001);
const data1 = eps_range.map(eps => ({eps, T: getTemp(F1, eps, channels[0])}));
const data2 = eps_range.map(eps => ({eps, T: getTemp(F2, eps, channels[1])}));
```

<div class="alg-figure">

```js
Plot.plot({
  width: 640,
  height: 400,
  marginLeft: 72,
  marginBottom: 44,
  grid: true,
  x: {label: "Emissivity ε"},
  y: {label: "Temperature (K)"},
  marks: [
    Plot.line(data1, {x: "eps", y: "T", stroke: "#e74c3c", strokeWidth: 2.5}),
    Plot.line(data2, {x: "eps", y: "T", stroke: "#3498db", strokeWidth: 2.5}),
    Plot.text([{x: 0.92, y: 295, text: "Channel 1"}], {
      x: "x", y: "y", text: "text", fill: "#e74c3c", fontSize: 14
    }),
    Plot.text([{x: 0.92, y: 285, text: "Channel 2"}], {
      x: "x", y: "y", text: "text", fill: "#3498db", fontSize: 14
    })
  ]
})
```

</div>

<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 1.5rem">Temperature-emissivity curves for two channels showing infinite solutions.</p>

<h3 style="margin:2rem 0 1rem">Modeling the Band-Pass Flux</h3>

<p>To enable efficient computation, the band-integrated blackbody flux is modeled as a second-order polynomial in temperature:</p>

<p>${tex.block`B_n(T) = a_n + b_n T + c_n T^2`}</p>

<p>This approximation is accurate to within 0.03 K over the range 265-310 K, making it suitable for Earth surface applications.</p>

```js
const channelIndex = view(Inputs.range([0, 5], {
  step: 1,
  value: 0,
  label: "Channel",
  format: i => channels[i].name
}));


<div class="alg-figure">


{
  const channel = channels[channelIndex];
  const T_range = d3.range(265, 310.5, 0.5);
  const data = T_range.map(T => ({T, B: B_n(T, channel)}));

  return Plot.plot({
    width: 640,
    height: 400,
    marginLeft: 72,
    marginBottom: 44,
    grid: true,
    x: {label: "Temperature (K)"},
    y: {label: "Band-pass Flux (W/cm²/sr)"},
    marks: [
      Plot.line(data, {x: "T", y: "B", stroke: "#2ecc71", strokeWidth: 2.5})
    ]
  });
}
```
```js
</div>

<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 1.5rem">Band-pass flux vs temperature with quadratic fit (TIMS channel simulation).</p>

<h3 style="margin:2rem 0 1rem">Temperature Bounds from Emissivity Constraints</h3>

<p>Given realistic emissivity bounds ${tex`\varepsilon_{n,\text{min}} \leq \varepsilon_n \leq \varepsilon_{n,\text{max}}`}, we can derive temperature bounds for each channel. Since temperature and emissivity are inversely related:</p>

<p>${tex.block`T_{n,\text{max}} = \delta(F_n, \varepsilon_{n,\text{min}}) > T > \delta(F_n, \varepsilon_{n,\text{max}}) = T_{n,\text{min}}`}</p>

<p>where ${tex`\delta`} is the inverse function relating flux and emissivity to temperature:</p>

<p>${tex.block`T = \delta(F_n, \varepsilon_n) = \frac{-b_n + \sqrt{b_n^2 - 4c_n(a_n - F_n/\varepsilon_n)}}{2c_n}`}</p>

```js
const eps1_min = view(Inputs.range([0.8, 1.0], {
  step: 0.01,
  value: 0.97,
  label: "ε₁ min"
}));
```

```js
const eps1_max = view(Inputs.range([0.8, 1.0], {
  step: 0.01,
  value: 1.00,
  label: "ε₁ max"
}));
```

```js
const eps2_min = view(Inputs.range([0.8, 1.0], {
  step: 0.01,
  value: 0.97,
  label: "ε₂ min"
}));
```

```js
const eps2_max = view(Inputs.range([0.8, 1.0], {
  step: 0.01,
  value: 1.00,
  label: "ε₂ max"
}));
```

<div class="alg-figure">

```js
{
  const T_true = 278;
  const eps1_true = 0.985;
  const eps2_true = 0.980;
  const F1 = eps1_true * B_n(T_true, channels[0]);
  const F2 = eps2_true * B_n(T_true, channels[1]);

  const eps_range = d3.range(0.8, 1.001, 0.002);
  const data1 = eps_range.map(eps => ({eps, T: getTemp(F1, eps, channels[0])}));
  const data2 = eps_range.map(eps => ({eps, T: getTemp(F2, eps, channels[1])}));

  // Calculate bounds
  const T1_max = getTemp(F1, eps1_min, channels[0]);
  const T1_min = getTemp(F1, eps1_max, channels[0]);
  const T2_max = getTemp(F2, eps2_min, channels[1]);
  const T2_min = getTemp(F2, eps2_max, channels[1]);

  const T_final_min = Math.max(T1_min, T2_min);
  const T_final_max = Math.min(T1_max, T2_max);
  const T_estimate = (T_final_min + T_final_max) / 2;
  const T_error = (T_final_max - T_final_min) / 2;

  return html`
    ${Plot.plot({
      width: 640,
      height: 450,
      marginLeft: 72,
      marginBottom: 44,
      grid: true,
      x: {label: "Emissivity ε", domain: [0.8, 1.0]},
      y: {label: "Temperature (K)"},
      marks: [
        Plot.line(data1, {x: "eps", y: "T", stroke: "#e74c3c", strokeWidth: 2.5}),
        Plot.line(data2, {x: "eps", y: "T", stroke: "#3498db", strokeWidth: 2.5}),
        Plot.rect([{x1: eps1_min, x2: eps1_max, y1: T1_min, y2: T1_max}], {
          x1: "x1", x2: "x2", y1: "y1", y2: "y2",
          fill: "#e74c3c", fillOpacity: 0.15
        }),
        Plot.rect([{x1: eps2_min, x2: eps2_max, y1: T2_min, y2: T2_max}], {
          x1: "x1", x2: "x2", y1: "y1", y2: "y2",
          fill: "#3498db", fillOpacity: 0.15
        }),
        Plot.ruleY([T_final_min, T_final_max], {
          stroke: "#27ae60", strokeWidth: 2.5, strokeDasharray: "6,4"
        }),
        Plot.text([{x: 0.85, y: 295, text: "Channel 1"}], {
          x: "x", y: "y", text: "text", fill: "#e74c3c", fontSize: 14
        }),
        Plot.text([{x: 0.85, y: 285, text: "Channel 2"}], {
          x: "x", y: "y", text: "text", fill: "#3498db", fontSize: 14
        })
      ]
    })}
    <div style="margin-top: 1rem; padding: 1rem; background: #e8f4f8; border-radius: 6px; border-left: 4px solid #27ae60;">
      <div style="font-weight: 600; margin-bottom: 0.5rem; color: #1a5490; font-family: var(--sans-serif);">Final Temperature Bounds:</div>
      <div style="font-size: 1.05rem; font-family: var(--serif);">
        <strong>T<sub>min</sub></strong> = ${T_final_min.toFixed(2)} K, 
        <strong>T<sub>max</sub></strong> = ${T_final_max.toFixed(2)} K<br>
        <strong>Estimate:</strong> T̂ = ${T_estimate.toFixed(2)} K ± ${T_error.toFixed(2)} K
      </div>
    </div>
  `;
}
```

</div>

<h4 style="margin:1rem 0 0.5rem">The Overlapping Region Principle</h4>

<p>With N channels, we obtain N independent sets of temperature bounds. The true temperature must satisfy all constraints simultaneously, so we take the intersection:</p>

<p>${tex.block`T_{\text{min}} = \max_n\{T_{n,\text{min}}\} \quad \text{and} \quad T_{\text{max}} = \min_n\{T_{n,\text{max}}\}`}</p>

<p>The final temperature estimate is the midpoint of these bounds:</p>

<p>${tex.block`\hat{T} = \frac{T_{\text{min}} + T_{\text{max}}}{2}`}</p>

<p>with maximum error:</p>

<p>${tex.block`\Delta T = \frac{T_{\text{max}} - T_{\text{min}}}{2}`}</p>

<h4 style="margin:1rem 0 0.5rem">Refining Emissivity Bounds</h4>

<p>Once we have tight temperature bounds, we refine the emissivity bounds for each channel:</p>

<p>${tex.block`\varepsilon_{n,\text{refined,min}} = \frac{F_n}{B_n(T_{\text{max}})} \quad \text{and} \quad \varepsilon_{n,\text{refined,max}} = \frac{F_n}{B_n(T_{\text{min}})}`}</p>

<p>The emissivity estimate is taken at the midpoint temperature:</p>

<p>${tex.block`\hat{\varepsilon}_n = \frac{F_n}{B_n(\hat{T})}`}</p>

<h3 style="margin:2rem 0 1rem">Application: Utah Lake Water Analysis</h3>

<p>The method was validated using TIMS data from Utah Lake (April 1991). For water pixels with initial emissivity bounds of 0.97–1.00 across all six channels:</p>

<div class="alg-figure">

```js
{
  const temps_bounds = Array.from({length: 100}, () => 276 + Math.random() * 4);
  const temps_brightness = Array.from({length: 100}, () => 275 + Math.random() * 5);

  const data = [
    ...temps_bounds.map(T => ({T, method: "Bounds Method"})),
    ...temps_brightness.map(T => ({T, method: "Brightness Temp"}))
  ];

  return Plot.plot({
    width: 640,
    height: 400,
    marginLeft: 72,
    marginBottom: 44,
    grid: true,
    x: {label: "Temperature (K)"},
    y: {label: "Frequency"},
    color: {legend: true},
    marks: [
      Plot.rectY(data, Plot.binX({y: "count"}, {
        x: "T",
        fill: "method",
        mixBlendMode: "multiply",
        thresholds: 20
      }))
    ]
  });
}
```

</div>

<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 1.5rem">Temperature distribution from bounds method vs brightness temperature (simulated).</p>

<div style="background: #f0f7ff; padding: 1.5rem; border-radius: 8px; margin: 2rem 0; box-shadow: 0 2px 8px rgba(0,0,0,0.1);">
  <h4 style="margin-top: 0; margin-bottom: 1rem; font-family: var(--sans-serif);">Key Results</h4>
  <ul style="margin: 0.5rem 0 0 1.5rem;">
    <li style="margin: 0.35rem 0;"><strong>Temperature accuracy</strong>: ±0.5°C average error</li>
    <li style="margin: 0.35rem 0;"><strong>Emissivity accuracy</strong>: ±0.0092 average error</li>
    <li style="margin: 0.35rem 0;">No assumed constant emissivity across channels</li>
    <li style="margin: 0.35rem 0;">No iterative optimization required</li>
    <li style="margin: 0.35rem 0;">Computationally efficient for large datasets</li>
  </ul>
</div>

<h3 style="margin:2rem 0 1rem">Advantages and Limitations</h3>

<h4 style="margin:1rem 0 0.5rem">Advantages</h4>
<ul style="margin: 0.5rem 0 0 1.5rem;">
  <li style="margin: 0.35rem 0;"><strong>No emissivity assumptions</strong>: Unlike single-channel methods, no channel requires assumed emissivity</li>
  <li style="margin: 0.35rem 0;"><strong>Physically realistic</strong>: Bounds reflect true measurement constraints</li>
  <li style="margin: 0.35rem 0;"><strong>Computationally simple</strong>: No iterative solvers or optimization</li>
  <li style="margin: 0.35rem 0;"><strong>Error quantification</strong>: Inherent uncertainty bounds provided</li>
</ul>

<h4 style="margin:1rem 0 0.5rem">Limitations</h4>
<ul style="margin: 0.5rem 0 0 1.5rem;">
  <li style="margin: 0.35rem 0;"><strong>Requires realistic initial bounds</strong>: Method depends on prior knowledge of material properties</li>
  <li style="margin: 0.35rem 0;"><strong>Bound tightness varies</strong>: Some materials provide tighter constraints than others</li>
  <li style="margin: 0.35rem 0;"><strong>Multi-channel requirement</strong>: Needs multiple spectral channels for effective constraint</li>
</ul>

</section>

<section class="alg-section alg-refs alg-narrow">
  <h3>References</h3>
  <ol>
    <li>Jaggi, S., Quattrochi, D., & Baskin, R. (1992). "An algorithm for the estimation of upper and lower bounds of the emissivity and temperature of a target from thermal multispectral airborne remotely sensed data." SPIE Vol. 1699, 420-427.</li>
    <li>Palluconi, F.D., & Meeks, G.R. (1985). "Thermal Infrared Multispectral Scanner (TIMS): An investigator's guide to TIMS data." NASA JPL Publication 85-32.</li>
    <li>Gillespie, A.R. (1986). "Enhancement of TIMS images for photointerpretation." Proceedings of the TIMS Data User's Workshop, NASA JPL Publication 86-38, 12-24.</li>
  </ol>
</section>

</div>


corrige esse codigo ese esta bien pero los plits me dice como SyntaxError: 'return' outside of function (23:2)
