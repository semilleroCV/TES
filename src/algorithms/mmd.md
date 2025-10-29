---
toc: false
---


<link rel="stylesheet" href="/algorithms/algorithm.css">

<div>
<a href="/" class="alg-back" aria-label="Back to home">←</a>
</div>

<div class="alg-container">
  <header class="alg-hero">
    <h1>MMD</h1>
    <hr>
    <p>Mean Maximum-Minimum Difference approach for TES.</p>
  </header>

  <section class="alg-meta">
    <div><strong>Author</strong>: Tsuneo Matsunaga</div>
    <span class="sep"></span>
    <div><strong>Date</strong>: 1994</div>
  </section> 

  <section class="alg-section alg-narrow">
    <p>The <strong>MMD</strong> method is a simpler adaptation of <strong>ADE</strong> method. The <strong>MMD</strong> utilizes the relationship between ${tex`\boldsymbol{\={\varepsilon}}`} and the total range of emissivities themselves, the maximum-minimum difference (<strong>MMD</strong>). It is assumed that ${tex`\boldsymbol{\={\varepsilon}}`} is a linear function of <strong>MMD</strong>.</p>
<div class="alg-figure" style="max-width:680px;margin:0.5rem auto 0">

```js
(() => {
  const regressionData = Array.from({length: 35}, (_, i) => {
    const MMD = 0.01 + 0.04 * i / 20;
    const epsilonMean = 0.96 - 0.5 * MMD + (Math.random() - 0.5) * 0.01;
    return {MMD, epsilonMean};
  });

  const fitLine = Array.from({length: 36.5}, (_, i) => {
    const MMD = i * 0.002 + 0.01;
    return {MMD, epsilonMean: 0.96 - 0.5 * MMD};
  });

  return Plot.plot({
    height: 220,
    marginLeft: 70,
    marginBottom: 40,
    grid: true,
    x: {label: "MMD"},
    y: {label: "Mean emissivity (ε̄)", domain: [0.92, 0.97]},
    marks: [
      Plot.dot(regressionData, {x: "MMD", y: "epsilonMean", fill: "#777"}),
      Plot.line(fitLine, {x: "MMD", y: "epsilonMean", stroke: "orange", strokeWidth: 2})
    ]
  });
})()

```
<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 0">Example: linear relationship between ${tex`\={\varepsilon}`} and MMD.</p>
</div>
    <p>The algorithm requires that the ${tex`\={\varepsilon}`} spectrum be estimated in order to calculate the <strong>MMD</strong>, from which ${tex`\boldsymbol{\={\varepsilon}}`} is calculated. The algorithm is implemented in five steps:</p>
  </section>

  <section class="alg-section">
    <h2>Steps</h2>
    <ol>
<li>The "first-guess" emissivities are calculated from surface radiance data using the <strong>NEM</strong> approach (in the figure you can see an example of an emissivity spectra "calculated").
<div class="alg-figure" style="max-width:720px;margin:0.5rem auto 0">

```js
(async () => {
  const lwirDomain = [8, 15];
  const samples = 240;
  const lambdas = Array.from({length: samples}, (_, i) =>
    lwirDomain[0] + (lwirDomain[1] - lwirDomain[0]) * (i / (samples - 1))
  );

  const epsNEM = lambdas.map(lamb =>
    0.9 - 0.01 * Math.sin((lamb - 8) * 1.3)
  ); 

  const data = lambdas.map((lamb, i) => ({
      band: lamb,
      eps: epsNEM[i]
    }));

  return Plot.plot({
    height: 260,
    marginLeft: 70,
    marginBottom: 40,
    grid: true,
    x: {label: "Longitud de onda (µm)", domain: lwirDomain},
    y: {label: "ε (emisividad)", domain: [0.87, 0.95]},
    marks: [
      Plot.line(data, {x: "band", y: "eps", stroke: "#d62728", strokeWidth: 2}),
      Plot.dot(data.filter((_, i) => i % 30 === 0), {x: "band", y: "eps", fill:"#d62728"})
    ]
  });
})()
```
<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 0">Example: emissivity initial spectra (NEM).</p>
</div>
</li>    
<li><p><strong>MMD</strong> is calculated from the "first-guess" emissivities or, on subsequent iterations, adjusted emissivities (as you can see in the figure):${tex.block`MMD_j = \varepsilon_{max,j}-\varepsilon_{min,j}`}
Where ${tex`\varepsilon_{max,j}`} and ${tex`\varepsilon_{min,j}`} are respectively the maximum and minimum ${tex`\varepsilon`} value, and ${tex`j`} indicates the current iteration.</p>
<div class="alg-figure" style="max-width:720px;margin:0.5rem auto 0">

```js
(async () => {
  const lwirDomain = [8, 15];
  const samples = 240;
  const lambdas = Array.from({length: samples}, (_, i) =>
    lwirDomain[0] + (lwirDomain[1] - lwirDomain[0]) * (i / (samples - 1))
  );

  const epsNEM = lambdas.map(lamb =>
    0.9 - 0.01 * Math.sin((lamb - 8) * 1.3)
  ); 

  const data = lambdas.map((lamb, i) => ({
      band: lamb,
      eps: epsNEM[i]
    }));

  const epsMax = Math.max(...data.map(d => d.eps));
  const epsMin = Math.min(...data.map(d => d.eps));
  const MMD = epsMax - epsMin;

  return Plot.plot({
    height: 260,
    marginLeft: 70,
    marginBottom: 40,
    grid: true,
    x: {label: "Longitud de onda (µm)", domain: lwirDomain},
    y: {label: "ε (emisividad)", domain: [0.87, 0.95]},
    marks: [
      Plot.areaY([{x: 10, y1: epsMax, y2: epsMin}, {x: 14, y1: epsMax, y2: epsMin}], 
                 {fill: "lightgray", opacity: 0.3}),
      Plot.line(data, {x: "band", y: "eps", stroke: "#d62728", strokeWidth: 2}),
      Plot.dot(data.filter((_, i) => i % 30 === 0), {x: "band", y: "eps", fill:"#d62728"}),
      Plot.ruleY([epsMax], {stroke: "orange", strokeDasharray: "4,2"}),
      Plot.ruleY([epsMin], {stroke: "blue", strokeDasharray: "4,2"})
    ]
  });
})()
```
<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 0">Example: MMD = εₘₐₓ - εₘᵢₙ = 0.9099 - 0.8900 = 0.0199.</p>
</div>
</li>
<li><p>The new mean emissivity, ${tex`\={\varepsilon}_{j+1}`} is calculated using the empirical equation: ${tex.block`\={\varepsilon}_{j+1} = a + b * MMD_j`} where <strong>a</strong> and <strong>b</strong> are constants determined by regression analyses of laboratory spectra of various terrestrial materials. The constant <strong>a</strong> may be interpreted as the emissivity of commonly occurring graybodies such as water, snow and vegetation.</p></li>
<li>The emissivities are then adjusted so that their average is equal to ${tex`\={\varepsilon}_{j+1}`} (allowing to adjust the amplitud whitout affecting its shape), with the equation: ${tex.block`\varepsilon_{i,j+1} = \varepsilon_{i,j} * (\={\varepsilon}_{j+1}/ \={\varepsilon}_{j})`} where ${tex`i`} is the band number.</li>
<li>${tex`T`} is calculated from ${tex`\varepsilon_{i,j+1}`} and ground-leaving radiances for each band</li>

<div class="alg-figure" style="max-width:720px;margin:0.5rem auto 0">

```js
(async () => {
  const lwirDomain = [8, 15];
  const samples = 240;
  const lambdas = Array.from({length: samples}, (_, i) =>
    lwirDomain[0] + (lwirDomain[1] - lwirDomain[0]) * (i / (samples - 1))
  );

  const epsReal = lambdas.map(λ =>
    0.98 - 0.015 * Math.exp(-((λ - 10.5) ** 2) / 1.2)
  );
  const epsNEM = lambdas.map(λ =>
    0.9 - 0.01 * Math.sin((λ - 8) * 1.3)
  );

  function epsAdjusted(iter) {
    const alpha = 1 - Math.exp(-0.35 * iter);
    return lambdas.map((λ, i) => epsNEM[i] + alpha * (epsReal[i] - epsNEM[i]));
  }

  const container = html`<div style="display:grid;gap:0.5rem;max-width:640px"></div>`;
  const slider = html`<input type="range" min="0" max="10" step="1" value="0" style="width:100%">`;
  const chartDiv = html`<div></div>`;
  container.append(chartDiv, slider);

  function update(iter) {
    const epsAdj = epsAdjusted(iter);
    const data = [
      ...lambdas.map((λ, i) => ({λ, ε: epsReal[i], tipo: "Real"})),
      ...lambdas.map((λ, i) => ({λ, ε: epsNEM[i], tipo: "NEM"})),
      ...lambdas.map((λ, i) => ({λ, ε: epsAdj[i], tipo: "Ajustado"}))
    ];

    const color = {
      "Real": "#1f77b4",
      "NEM": "#d62728",
      "Ajustado": "#2ca02c"
    };

    chartDiv.replaceChildren(Plot.plot({
      height: 260,
      marginLeft: 70,
      marginBottom: 40,
      grid: true,
      x: {label: "Longitud de onda (µm)", domain: lwirDomain},
      y: {label: "ε (emisividad)", domain: [0.85, 1]},
      color: {legend: true, domain: Object.keys(color), range: Object.values(color)},
      marks: [
        Plot.line(data, {x: "λ", y: "ε", stroke: "tipo", strokeWidth: 2}),
        Plot.dot(data.filter((_, i) => i % 30 === 0), {x: "λ", y: "ε", fill: "tipo"})
      ]
    }));
  }

  slider.addEventListener("input", () => update(+slider.value));
  update(0);

  return container;
})()
```
<p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 0">Example: spectral adjustment after iterations.</p>
</div>
</ol>
Steps 2-5 are repeated until ${tex`Ts_{j+1}-Ts_j < NE\Delta T`}.
</section>
  <section class="alg-section">
    <h3>Advantages and limitations</h3>
    <ul>
      <li><strong>T</strong> is relatively <strong>insensitive</strong> to multiple scattering of thermal radiation within a scene element and to downwelling sky irradiance.</li>
      <li>Calculated <strong>emissivities</strong> are more sensitive to multiple scattering of thermal radiation within a scene element and to downwelling sky irradiance.</li>
      <li>The accuracy of the method depends on the accuracy of the empirical relationship between ${tex`\={\varepsilon}`} and <strong>MMD</strong> and on ${tex`NE\Delta T`}.</li>
    </ul>
  </section>

  
  <section class="alg-section alg-narrow">
    <p>Here is an representation from the original paper that shows the relationship between ${tex`\={\varepsilon}`} and ${tex`MMD`} for various terrestrial materials.</p>
  </section>

  <div class="alg-figure" style="max-width:720px;margin:0.5rem auto 1.5rem">
    <img src="/algorithms/assets/mean_mmd.png" alt="Alpha Regression" style="width:100%;height:auto;">
  </div>

  <section class="alg-section alg-refs">
    <h3>References</h3>
    <ol>
      <li>Gillespie, Alan R., et al. "Temperature/emissivity separation algorithm theoretical basis document, version 2.4." ATBD contract NAS5-31372, NASA (1999).</li>
    </ol>
  </section>
</div>
