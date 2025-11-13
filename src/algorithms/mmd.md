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
    <p>The <strong>MMD</strong> method is a simpler adaptation of <strong>ADE</strong> method. The <strong>MMD</strong> utilizes the relationship between ${tex`\boldsymbol{\={\varepsilon}}`} and the total range of emissivities themselves, the maximum-minimum difference (<strong>MMD</strong>). It is assumed that ${tex`\boldsymbol{\={\varepsilon}}`} is a linear function of <strong>MMD</strong>. The intuition behind this algorithm is to preserve the emissivity (${tex`\varepsilon`}) amplitude.</p>
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
<li>The "first-guess" emissivities are calculated from surface radiance data using the <strong><a href='nem'>NEM</a></strong> approach (for the examples we will be working with a synthetic data of 5 points in the LWIR window. In the figure you can see an example of an emissivity spectra "calculated" using NEM).
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
<li><p>The new mean emissivity, ${tex`\={\varepsilon}_{j+1}`} is calculated using the empirical equation: ${tex.block`\={\varepsilon}_{j+1} = a + b * MMD_j`} where <strong>a</strong> and <strong>b</strong> are constants determined by regression analyses of laboratory spectra of various terrestrial materials. The constant <strong>a</strong> may be interpreted as the emissivity of commonly occurring graybodies such as water, snow and vegetation. In the next figure, you can see the raltionship between <strong>MMD</strong> and ${tex`\={\varepsilon}`} for various terrestrial materials.</p></li>

<div class="alg-figure" style="max-width:720px;margin:0.5rem auto 1.5rem">
    <img src="/algorithms/assets/mean_mmd.png" alt="Alpha Regression" style="width:100%;height:auto;">
  </div>

<li>The emissivities are then adjusted so that their average is equal to ${tex`\={\varepsilon}_{j+1}`} (allowing to adjust the amplitude of the emissivity (${tex`\varepsilon`}) calculated from <strong>NEM</strong> whitout affecting its shape), with the equation: ${tex.block`\varepsilon_{i,j+1} = \varepsilon_{i,j} * (\={\varepsilon}_{j+1}/ \={\varepsilon}_{j})`} where ${tex`i`} is the band number.</li>
<li>${tex`T`} is calculated from ${tex`\varepsilon_{i,j+1}`} and ground-leaving radiances for each band</li>

<div class="alg-figure" style="max-width:720px;margin:0.5rem auto 0">

```js
(async () => {
  const lwirDomain = [8, 15];
  const samples = 240;
  const maxIter = 10;
  const lambdas = Array.from({ length: samples }, (_, i) =>
    lwirDomain[0] + (lwirDomain[1] - lwirDomain[0]) * (i / (samples - 1))
  );

  // Emisividad "real" y "NEM"
  const epsReal = lambdas.map(
    (λ) => 0.98 - 0.015 * Math.exp(-((λ - 10.5) ** 2) / 1.2)
  );
  const epsNEM = lambdas.map((λ) => 0.9 - 0.01 * Math.sin((λ - 8) * 1.3));

  // Función de ajuste
  function epsAdjusted(iter) {
    const alpha = 1 - Math.exp(-0.35 * iter);
    return lambdas.map((λ, i) => epsNEM[i] + alpha * (epsReal[i] - epsNEM[i]));
  }

  // Funciones auxiliares
  const mean = (arr) => arr.reduce((a, b) => a + b, 0) / arr.length;
  const max = (arr) => Math.max(...arr);
  const min = (arr) => Math.min(...arr);

  // Contenedor principal
  const container = html`<div style="display:grid;gap:0.75rem;max-width:720px"></div>`;
  const slider = html`<input type="range" min="0" max="${maxIter}" step="1" value="0" style="width:100%">`;
  const chartDiv = html`<div></div>`;
  const mmdChartDiv = html`<div></div>`;
  const infoDiv = html`<div style="font-family:monospace;font-size:0.9rem;padding:0.5rem;background:#f8f8f8;border-radius:8px;"></div>`;
  container.append(chartDiv, mmdChartDiv, slider, infoDiv);

  // Historial del MMD
  const mmdHistory = [];

  function update(iter) {
    const epsAdj = epsAdjusted(iter);

    // Calcular métricas
    const meanReal = mean(epsReal);
    const meanNEM = mean(epsNEM);
    const meanAdj = mean(epsAdj);

    const maxAdj = max(epsAdj);
    const minAdj = min(epsAdj);
    const mmd = maxAdj - minAdj;

    // Guardar MMD histórico si no está calculado
    mmdHistory[iter] = mmd;

    // Datos para el gráfico principal
    const data = [
      ...lambdas.map((λ, i) => ({ λ, ε: epsReal[i], tipo: "Real" })),
      ...lambdas.map((λ, i) => ({ λ, ε: epsNEM[i], tipo: "NEM" })),
      ...lambdas.map((λ, i) => ({ λ, ε: epsAdj[i], tipo: "Ajustado" })),
    ];

    const color = {
      Real: "#1f77b4",
      NEM: "#d62728",
      Ajustado: "#2ca02c",
    };

    // === Gráfico principal ===
    chartDiv.replaceChildren(
      Plot.plot({
        height: 260,
        marginLeft: 70,
        marginBottom: 40,
        grid: true,
        x: { label: "Longitud de onda (µm)", domain: lwirDomain },
        y: { label: "ε (emisividad)", domain: [0.85, 1] },
        color: {
          legend: true,
          domain: Object.keys(color),
          range: Object.values(color),
        },
        marks: [
          Plot.line(data, { x: "λ", y: "ε", stroke: "tipo", strokeWidth: 2 }),
          Plot.dot(
            data.filter((_, i) => i % 30 === 0),
            { x: "λ", y: "ε", fill: "tipo" }
          ),
        ],
      })
    );

    // === Gráfico del MMD por iteración ===
    const mmdData = mmdHistory
      .map((val, i) => ({ iter: i, MMD: val }))
      .filter((d) => d.MMD !== undefined);

    mmdChartDiv.replaceChildren(
      Plot.plot({
        height: 180,
        marginLeft: 60,
        marginBottom: 40,
        grid: true,
        x: { label: "Iteración", domain: [0, maxIter] },
        y: { label: "MMD (εmax - εmin)" },
        marks: [
          Plot.line(mmdData, { x: "iter", y: "MMD", stroke: "#ff7f0e", strokeWidth: 2 }),
          Plot.dot([{ iter, MMD: mmd }], { x: "iter", y: "MMD", fill: "#ff7f0e", r: 4 }),
        ],
      })
    );

    // === Información textual ===
    infoDiv.innerHTML = `
      <b>Iteración:</b> ${iter}<br>
      <b>Emisividad media:</b><br>
      • Real: ${meanReal.toFixed(4)}<br>
      • NEM: ${meanNEM.toFixed(4)}<br>
      • Ajustado: ${meanAdj.toFixed(4)}<br>
      <b>MMD actual (εmax - εmin):</b> ${mmd.toFixed(6)}
    `;
  }

  slider.addEventListener("input", (e) => update(+e.target.value));
  update(0);
  display(container);
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

  <section class="alg-section alg-refs">
    <h3>References</h3>
    <ol>
      <li>Gillespie, Alan R., et al. "Temperature/emissivity separation algorithm theoretical basis document, version 2.4." ATBD contract NAS5-31372, NASA (1999).</li>
      <li>Matsunaga, T., A Temperature-Emissivity Separation Method Using an Empirical Relationship between the Mean, the Maximum, and the Minimum of the Thermal Infrared Emissivity Spectrum. Jour. Remote Sens. Soc. Japan 14(2), 230-241, 1994 (in Japanese with English abstract).</li>
    </ol>
  </section>
</div>
