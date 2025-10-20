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

  <h3 style="margin:1rem 0 0.5rem">The Underdetermined Problem</h3>
    <p>For each channel <i>n</i>, the measured radiance ${tex`F_n`} relates to temperature ${tex`T`} and emissivity ${tex`\varepsilon_n`} through the band-integrated blackbody function:</p>

  <div class="alg-split">
    <div class="alg-formula">
      <p style="margin-top:0">${tex.block`F_n = \varepsilon_n B_n(T)`}</p>
      <p style="font-size:0.9rem;color:var(--theme-foreground-muted);margin-top:1rem">where ${tex`B_n(T)`} is the band-integrated Planck function for channel <i>n</i></p>
    </div>
    <div class="alg-figure bleed">

```js
(() => {
  // Constants for TIMS-like channels
  const channels = [
    {name: 'Channel 1', lambda: 8.3, a: 0.05, b: 0.003, c: 0.000001},
    {name: 'Channel 2', lambda: 8.65, a: 0.06, b: 0.0035, c: 0.0000012}
  ];

  // Band-integrated blackbody function (W/cm²/sr)
  function B_n(T, channel) {
    return channel.a + channel.b * T + channel.c * T * T;
  }

  // Inverse function: get T from F and epsilon
  function getTemp(F, eps, channel) {
    const a = channel.a - F / eps;
    const discriminant = channel.b * channel.b - 4 * channel.c * a;
    if (discriminant < 0) return null;
    return (-channel.b + Math.sqrt(discriminant)) / (2 * channel.c);
  }

  const T_true = 285; // True temperature
  const eps1_true = 0.98;
  const eps2_true = 0.97;
  const F1 = eps1_true * B_n(T_true, channels[0]);
  const F2 = eps2_true * B_n(T_true, channels[1]);

  const eps_range = [];
  const T1_range = [];
  const T2_range = [];

  for (let eps = 0.85; eps <= 1.0; eps += 0.001) {
    eps_range.push(eps);
    T1_range.push(getTemp(F1, eps, channels[0]));
    T2_range.push(getTemp(F2, eps, channels[1]));
  }

  const data1 = eps_range.map((eps, i) => ({eps, T: T1_range[i]}));
  const data2 = eps_range.map((eps, i) => ({eps, T: T2_range[i]}));

  return Plot.plot({
    height: 340,
    marginLeft: 60,
    marginBottom: 44,
    x: {label: "Emissivity ε", domain: [0.85, 1.0]},
    y: {label: "Temperature (K)", grid: true},
    marks: [
      Plot.line(data1, {x: "eps", y: "T", stroke: "#e74c3c", strokeWidth: 2.5}),
      Plot.line(data2, {x: "eps", y: "T", stroke: "#3498db", strokeWidth: 2.5}),
      Plot.text([{x: 0.95, y: 295, label: "Channel 1"}], {
        x: "x", y: "y", text: "label", fill: "#e74c3c", fontSize: 12
      }),
      Plot.text([{x: 0.88, y: 278, label: "Channel 2"}], {
        x: "x", y: "y", text: "label", fill: "#3498db", fontSize: 12
      }),
      Plot.ruleY([0])
    ]
  });
})()
```

  <p style="text-align:center;color:var(--theme-foreground-muted);margin:0.5rem 0 0;font-size:0.9rem">Temperature-emissivity curves for two channels showing infinite solutions</p>
    </div>
  </div>
  </section>

  <section class="alg-section alg-narrow">
    <h3>Modeling the Band-Pass Flux</h3>
    <p>To enable efficient computation, the band-integrated blackbody flux is modeled as a second-order polynomial in temperature:</p>
    <p>${tex.block`B_n(T) = a_n + b_n T + c_n T^2`}</p>
    <p>This approximation is accurate to within 0.03 K over the range 265-310 K, making it suitable for Earth surface applications.</p>

  <div class="alg-figure" style="max-width:720px;margin:1.5rem auto">

```js
(() => {
  const container = document.createElement("div");
  
  const channels = [
    {name: 'Channel 1', lambda: 8.3, a: 0.05, b: 0.003, c: 0.000001},
    {name: 'Channel 2', lambda: 8.65, a: 0.06, b: 0.0035, c: 0.0000012},
    {name: 'Channel 3', lambda: 9.1, a: 0.07, b: 0.004, c: 0.0000014},
    {name: 'Channel 4', lambda: 9.8, a: 0.08, b: 0.0045, c: 0.0000015},
    {name: 'Channel 5', lambda: 10.6, a: 0.09, b: 0.005, c: 0.0000016},
    {name: 'Channel 6', lambda: 11.2, a: 0.10, b: 0.0055, c: 0.0000017}
  ];

  function B_n(T, channel) {
    return channel.a + channel.b * T + channel.c * T * T;
  }

  function draw(channelIdx) {
    const channel = channels[channelIdx];
    const T_range = [];
    const B_values = [];

    for (let T = 265; T <= 310; T += 0.5) {
      T_range.push(T);
      B_values.push(B_n(T, channel));
    }

    const data = T_range.map((T, i) => ({T, B: B_values[i]}));

    return Plot.plot({
      height: 320,
      marginLeft: 80,
      marginBottom: 44,
      x: {label: "Temperature (K)", domain: [265, 310]},
      y: {label: "Band-pass Flux (W/cm²/sr)", grid: true},
      marks: [
        Plot.line(data, {x: "T", y: "B", stroke: "#2ecc71", strokeWidth: 2.5}),
        Plot.text([{x: 300, y: data[data.length - 10].B, label: channel.name}], {
          x: "x", y: "y", text: "label", fill: "#2ecc71", fontSize: 12, dy: -10
        }),
        Plot.ruleY([0])
      ]
    });
  }

  let currentChannel = 0;
  let plot = draw(currentChannel);
  container.append(plot);

  const controls = document.createElement("div");
  controls.style.display = "flex";
  controls.style.alignItems = "center";
  controls.style.gap = "0.5rem";
  controls.style.marginTop = "0.75rem";

  const label = document.createElement("label");
  label.textContent = "Channel:";
  label.style.minWidth = "80px";

  const slider = document.createElement("input");
  slider.type = "range";
  slider.min = "0";
  slider.max = "5";
  slider.step = "1";
  slider.value = String(currentChannel);
  slider.style.flex = "1 1 auto";

  const valueEl = document.createElement("span");
  valueEl.textContent = channels[currentChannel].name;
  valueEl.style.minWidth = "5rem";
  valueEl.style.fontWeight = "600";
  valueEl.style.color = "var(--theme-foreground)";

  controls.append(label, slider, valueEl);
  container.append(controls);

  slider.addEventListener('input', () => {
    currentChannel = Number(slider.value);
    valueEl.textContent = channels[currentChannel].name;
    const next = draw(currentChannel);
    plot.replaceWith(next);
    plot = next;
  });

  return container;
})()
```

  <p style="text-align:center;color:var(--theme-foreground-muted);margin:1rem 0 0;font-size:0.9rem">Band-pass flux vs temperature with quadratic fit (TIMS channel simulation)</p>
  </div>
  </section>

  <section class="alg-section alg-narrow">
    <h3>Temperature Bounds from Emissivity Constraints</h3>
    <p>Given realistic emissivity bounds ${tex`\varepsilon_{n,\text{min}} \leq \varepsilon_n \leq \varepsilon_{n,\text{max}}`}, we can derive temperature bounds for each channel. Since temperature and emissivity are inversely related:</p>
    
    <p>${tex.block`T_{n,\text{max}} = \delta(F_n, \varepsilon_{n,\text{min}}) > T > \delta(F_n, \varepsilon_{n,\text{max}}) = T_{n,\text{min}}`}</p>
    
    <p>where ${tex`\delta`} is the inverse function relating flux and emissivity to temperature:</p>
    
    <p>${tex.block`T = \delta(F_n, \varepsilon_n) = \frac{-b_n + \sqrt{b_n^2 - 4c_n(a_n - F_n/\varepsilon_n)}}{2c_n}`}</p>

  <div class="alg-figure" style="max-width:100%;margin:1.5rem auto">

```js
(() => {
  const container = document.createElement("div");
  
  const channels = [
    {name: 'Channel 1', lambda: 8.3, a: 0.05, b: 0.003, c: 0.000001},
    {name: 'Channel 2', lambda: 8.65, a: 0.06, b: 0.0035, c: 0.0000012}
  ];

  function B_n(T, channel) {
    return channel.a + channel.b * T + channel.c * T * T;
  }

  function getTemp(F, eps, channel) {
    const a = channel.a - F / eps;
    const discriminant = channel.b * channel.b - 4 * channel.c * a;
    if (discriminant < 0) return null;
    return (-channel.b + Math.sqrt(discriminant)) / (2 * channel.c);
  }

  const T_true = 278;
  const eps1_true = 0.985;
  const eps2_true = 0.980;
  const F1 = eps1_true * B_n(T_true, channels[0]);
  const F2 = eps2_true * B_n(T_true, channels[1]);

  function draw(eps1_min, eps1_max, eps2_min, eps2_max) {
    const eps_range = [];
    const T1_curve = [];
    const T2_curve = [];

    for (let eps = 0.8; eps <= 1.0; eps += 0.002) {
      eps_range.push(eps);
      T1_curve.push(getTemp(F1, eps, channels[0]));
      T2_curve.push(getTemp(F2, eps, channels[1]));
    }

    const data1 = eps_range.map((eps, i) => ({eps, T: T1_curve[i]}));
    const data2 = eps_range.map((eps, i) => ({eps, T: T2_curve[i]}));

    // Calculate bounds
    const T1_max = getTemp(F1, eps1_min, channels[0]);
    const T1_min = getTemp(F1, eps1_max, channels[0]);
    const T2_max = getTemp(F2, eps2_min, channels[1]);
    const T2_min = getTemp(F2, eps2_max, channels[1]);

    const T_final_min = Math.max(T1_min, T2_min);
    const T_final_max = Math.min(T1_max, T2_max);

    // Create shaded regions
    const region1 = [];
    const region2 = [];
    
    for (let eps = eps1_min; eps <= eps1_max; eps += 0.01) {
      const T = getTemp(F1, eps, channels[0]);
      if (T >= T1_min && T <= T1_max) {
        region1.push({eps, T});
      }
    }

    for (let eps = eps2_min; eps <= eps2_max; eps += 0.01) {
      const T = getTemp(F2, eps, channels[1]);
      if (T >= T2_min && T <= T2_max) {
        region2.push({eps, T});
      }
    }

    const plot = Plot.plot({
      height: 400,
      marginLeft: 60,
      marginBottom: 44,
      x: {label: "Emissivity ε", domain: [0.8, 1.0]},
      y: {label: "Temperature (K)", grid: true},
      marks: [
        // Shaded bounds regions
        Plot.dot(region1, {x: "eps", y: "T", fill: "#e74c3c", opacity: 0.1, r: 2}),
        Plot.dot(region2, {x: "eps", y: "T", fill: "#3498db", opacity: 0.1, r: 2}),
        // Main curves
        Plot.line(data1, {x: "eps", y: "T", stroke: "#e74c3c", strokeWidth: 2.5}),
        Plot.line(data2, {x: "eps", y: "T", stroke: "#3498db", strokeWidth: 2.5}),
        // Final temperature bounds (horizontal lines)
        Plot.ruleY([T_final_min], {stroke: "#27ae60", strokeWidth: 2, strokeDasharray: "5,5"}),
        Plot.ruleY([T_final_max], {stroke: "#27ae60", strokeWidth: 2, strokeDasharray: "5,5"}),
        // Labels
        Plot.text([{x: 0.82, y: T_final_min, label: `T_min = ${T_final_min.toFixed(1)} K`}], {
          x: "x", y: "y", text: "label", fill: "#27ae60", fontSize: 11, dy: 12
        }),
        Plot.text([{x: 0.82, y: T_final_max, label: `T_max = ${T_final_max.toFixed(1)} K`}], {
          x: "x", y: "y", text: "label", fill: "#27ae60", fontSize: 11, dy: -8
        }),
        Plot.ruleY([0])
      ]
    });

    return {plot, T_final_min, T_final_max};
  }

  let eps1_min = 0.97, eps1_max = 1.0;
  let eps2_min = 0.97, eps2_max = 1.0;
  let result = draw(eps1_min, eps1_max, eps2_min, eps2_max);
  container.append(result.plot);

  // Controls for epsilon 1
  const controls1 = document.createElement("div");
  controls1.style.display = "flex";
  controls1.style.alignItems = "center";
  controls1.style.gap = "0.5rem";
  controls1.style.marginTop = "0.75rem";

  const label1 = document.createElement("label");
  label1.textContent = "ε₁ bounds:";
  label1.style.minWidth = "100px";

  const slider1min = document.createElement("input");
  slider1min.type = "range";
  slider1min.min = "0.8";
  slider1min.max = "1.0";
  slider1min.step = "0.01";
  slider1min.value = String(eps1_min);
  slider1min.style.flex = "1";

  const val1min = document.createElement("span");
  val1min.textContent = eps1_min.toFixed(2);
  val1min.style.minWidth = "3rem";
  val1min.style.fontSize = "0.9rem";

  const sep1 = document.createElement("span");
  sep1.textContent = "—";
  sep1.style.margin = "0 0.25rem";

  const slider1max = document.createElement("input");
  slider1max.type = "range";
  slider1max.min = "0.8";
  slider1max.max = "1.0";
  slider1max.step = "0.01";
  slider1max.value = String(eps1_max);
  slider1max.style.flex = "1";

  const val1max = document.createElement("span");
  val1max.textContent = eps1_max.toFixed(2);
  val1max.style.minWidth = "3rem";
  val1max.style.fontSize = "0.9rem";

  controls1.append(label1, slider1min, val1min, sep1, slider1max, val1max);

  // Controls for epsilon 2
  const controls2 = document.createElement("div");
  controls2.style.display = "flex";
  controls2.style.alignItems = "center";
  controls2.style.gap = "0.5rem";
  controls2.style.marginTop = "0.5rem";

  const label2 = document.createElement("label");
  label2.textContent = "ε₂ bounds:";
  label2.style.minWidth = "100px";

  const slider2min = document.createElement("input");
  slider2min.type = "range";
  slider2min.min = "0.8";
  slider2min.max = "1.0";
  slider2min.step = "0.01";
  slider2min.value = String(eps2_min);
  slider2min.style.flex = "1";

  const val2min = document.createElement("span");
  val2min.textContent = eps2_min.toFixed(2);
  val2min.style.minWidth = "3rem";
  val2min.style.fontSize = "0.9rem";

  const sep2 = document.createElement("span");
  sep2.textContent = "—";
  sep2.style.margin = "0 0.25rem";

  const slider2max = document.createElement("input");
  slider2max.type = "range";
  slider2max.min = "0.8";
  slider2max.max = "1.0";
  slider2max.step = "0.01";
  slider2max.value = String(eps2_max);
  slider2max.style.flex = "1";

  const val2max = document.createElement("span");
  val2max.textContent = eps2_max.toFixed(2);
  val2max.style.minWidth = "3rem";
  val2max.style.fontSize = "0.9rem";

  controls2.append(label2, slider2min, val2min, sep2, slider2max, val2max);

  // Results display
  const resultsDiv = document.createElement("div");
  resultsDiv.style.marginTop = "1rem";
  resultsDiv.style.padding = "1rem";
  resultsDiv.style.background = "#e8f4f8";
  resultsDiv.style.borderRadius = "4px";

  const resultsTitle = document.createElement("div");
  resultsTitle.style.fontWeight = "600";
  resultsTitle.style.marginBottom = "0.5rem";
  resultsTitle.textContent = "Final Temperature Bounds:";

  const resultsText = document.createElement("div");
  resultsText.style.fontSize = "1.1rem";
  resultsText.style.fontVariantNumeric = "tabular-nums";
  const T_est = (result.T_final_min + result.T_final_max) / 2;
  const T_err = (result.T_final_max - result.T_final_min) / 2;
  resultsText.innerHTML = `<strong>T<sub>min</sub></strong> = ${result.T_final_min.toFixed(2)} K, <strong>T<sub>max</sub></strong> = ${result.T_final_max.toFixed(2)} K<br>` +
    `<strong>Estimate:</strong> T̂ = ${T_est.toFixed(2)} K ± ${T_err.toFixed(2)} K`;

  resultsDiv.append(resultsTitle, resultsText);

  container.append(controls1, controls2, resultsDiv);

  function updatePlot() {
    eps1_min = parseFloat(slider1min.value);
    eps1_max = parseFloat(slider1max.value);
    eps2_min = parseFloat(slider2min.value);
    eps2_max = parseFloat(slider2max.value);

    val1min.textContent = eps1_min.toFixed(2);
    val1max.textContent = eps1_max.toFixed(2);
    val2min.textContent = eps2_min.toFixed(2);
    val2max.textContent = eps2_max.toFixed(2);

    const newResult = draw(eps1_min, eps1_max, eps2_min, eps2_max);
    result.plot.replaceWith(newResult.plot);
    result = newResult;

    const T_est = (result.T_final_min + result.T_final_max) / 2;
    const T_err = (result.T_final_max - result.T_final_min) / 2;
    resultsText.innerHTML = `<strong>T<sub>min</sub></strong> = ${result.T_final_min.toFixed(2)} K, <strong>T<sub>max</sub></strong> = ${result.T_final_max.toFixed(2)} K<br>` +
      `<strong>Estimate:</strong> T̂ = ${T_est.toFixed(2)} K ± ${T_err.toFixed(2)} K`;
  }

  slider1min.addEventListener('input', updatePlot);
  slider1max.addEventListener('input', updatePlot);
  slider2min.addEventListener('input', updatePlot);
  slider2max.addEventListener('input', updatePlot);

  return container;
})()
```

  </div>
  </section>

  <section class="alg-section alg-narrow">
    <h4>The Overlapping Region Principle</h4>
    <p>With N channels, we obtain N independent sets of temperature bounds. The true temperature must satisfy all constraints simultaneously, so we take the intersection:</p>
    
    <p>${tex.block`T_{\text{min}} = \max_n\{T_{n,\text{min}}\} \quad \text{and} \quad T_{\text{max}} = \min_n\{T_{n,\text{max}}\}`}</p>
    
    <p>The final temperature estimate is the midpoint of these bounds:</p>
    
    <p>${tex.block`\hat{T} = \frac{T_{\text{min}} + T_{\text{max}}}{2}`}</p>
    
    <p>with maximum error:</p>
    
    <p>${tex.block`\Delta T = \frac{T_{\text{max}} - T_{\text{min}}}{2}`}</p>

  <h4 style="margin:1.5rem 0 0.75rem">Refining Emissivity Bounds</h4>
    <p>Once we have tight temperature bounds, we refine the emissivity bounds for each channel:</p>
    
    <p>${tex.block`\varepsilon_{n,\text{refined,min}} = \frac{F_n}{B_n(T_{\text{max}})} \quad \text{and} \quad \varepsilon_{n,\text{refined,max}} = \frac{F_n}{B_n(T_{\text{min}})}`}</p>
    
    <p>The emissivity estimate is taken at the midpoint temperature:</p>
    
    <p>${tex.block`\hat{\varepsilon}_n = \frac{F_n}{B_n(\hat{T})}`}</p>
  </section>

  <section class="alg-section alg-narrow">
    <h3>Application: Utah Lake Water Analysis</h3>
    <p>The method was validated using TIMS data from Utah Lake (April 1991). For water pixels with initial emissivity bounds of 0.97–1.00 across all six channels:</p>
    
  <div class="alg-figure" style="max-width:720px;margin:1.5rem auto">

```js
(() => {
  // Simulated temperature distributions
  const temps_bounds = [];
  const temps_brightness = [];
  
  for (let i = 0; i < 100; i++) {
    temps_bounds.push(276 + Math.random() * 4);
    temps_brightness.push(275 + Math.random() * 5);
  }

  const data_bounds = temps_bounds.map(t => ({temp: t, method: 'Bounds Method'}));
  const data_bright = temps_brightness.map(t => ({temp: t, method: 'Brightness Temp'}));

  return Plot.plot({
    height: 320,
    marginLeft: 60,
    marginBottom: 44,
    x: {label: "Temperature (K)", domain: [273, 283]},
    y: {label: "Frequency", grid: true},
    color: {legend: true, domain: ['Bounds Method', 'Brightness Temp'], range: ['#2ecc71', '#e67e22']},
    marks: [
      Plot.rectY(data_bounds, Plot.binX({y: "count"}, {x: "temp", fill: "#2ecc71", fillOpacity: 0.7})),
      Plot.rectY(data_bright, Plot.binX({y: "count"}, {x: "temp", fill: "#e67e22", fillOpacity: 0.7}))
    ]
  });
})()
```

  <p style="text-align:center;color:var(--theme-foreground-muted);margin:1rem 0 0;font-size:0.9rem">Temperature distribution from bounds method vs brightness temperature (simulated)</p>
  </div>

  <div style="background:#f0f7ff;padding:1.5rem;border-radius:8px;margin:2rem 0">
    <h4 style="margin-top:0">Key Results</h4>
    <ul style="margin:0.5rem 0">
      <li>Temperature accuracy: ±0.5°C average error</li>
      <li>Emissivity accuracy: ±0.0092 average error</li>
      <li>No assumed constant emissivity across channels</li>
      <li>No iterative optimization required</li>
      <li>Computationally efficient for large datasets</li>
    </ul>
  </div>
  </section>

  <section class="alg-section alg-narrow">
    <h3>Advantages and Limitations</h3>
    
    <h4>Advantages</h4>
    <ul>
      <li><strong>No emissivity assumptions:</strong> Unlike single-channel methods, no channel requires assumed emissivity</li>
      <li><strong>Physically realistic:</strong> Bounds reflect true measurement constraints</li>
      <li><strong>Computationally simple:</strong> No iterative solvers or optimization</li>
      <li><strong>Error quantification:</strong> Inherent uncertainty bounds provided</li>
    </ul>

    <h4>Limitations</h4>
    <ul>
      <li><strong>Requires realistic initial bounds:</strong> Method depends on prior knowledge of material properties</li>
      <li><strong>Bound tightness varies:</strong> Some materials provide tighter constraints than others</li>
      <li><strong>Multi-channel requirement:</strong> Needs multiple spectral channels for effective constraint</li>
    </ul>
  </section>

  <section class="alg-section alg-refs">
    <h3>References</h3>
    <ol>
      <li>Jaggi, S., Quattrochi, D., & Baskin, R. (1992). "An algorithm for the estimation of upper and lower bounds of the emissivity and temperature of a target from thermal multispectral airborne remotely sensed data." SPIE Vol. 1699, 420-427.</li>
      <li>Palluconi, F.D., & Meeks, G.R. (1985). "Thermal Infrared Multispectral Scanner (TIMS): An investigator's guide to TIMS data." NASA JPL Publication 85-32.</li>
      <li>Gillespie, A.R. (1986). "Enhancement of TIMS images for photointerpretation." Proceedings of the TIMS Data User's Workshop, NASA JPL Publication 86-38, 12-24.</li>
    </ol>
  </section>
</div>
