---
toc: false
---

<link rel="stylesheet" href="/algorithms/algorithm.css">


<div>
<a href="/" class="alg-back" aria-label="Back to home">←</a>
</div>

<div class="alg-container">
  <header class="alg-hero">
    <h1>ADE</h1>
    <p>Alpha Derived Emissivity.</p>
  </header>

  <section class="alg-meta">
    <div><strong>Authors</strong>: Jane Doe, John Roe</div>
    <span class="sep"></span>
    <div><strong>Date</strong>: 1990</div>
  </section>

  <section class="alg-section alg-narrow">
    <p>The ADE method rests on two key facts: temperature is constant across wavelength (it does not vary with ${tex`\lambda`}) while emissivity typically does; and it leverages Wien's approximation of the blackbody to relate average emissivity to temperature‑induced variance via regression. First, we ground ourselves in the exact blackbody law.</p>

  <h3 style="margin:1rem 0 0.5rem">Planck’s law</h3>
    <p>Planck’s law gives the spectral radiance \(B_\lambda(T)\) of an ideal blackbody as a function of wavelength and temperature.</p>

  <div class="alg-split">
    <div class="alg-formula">
      <p style="margin-top:0">${tex.block`B_\lambda(T)=\frac{2hc^2}{\lambda^5}\,\frac{1}{\exp\!\left(\tfrac{hc}{\lambda kT}\right)-1}`}</p>
    </div>
    <div class="alg-figure bleed">



```js
(async () => {
  const container = document.createElement("div");

  const samples = 600;
  const c = 299792458, h = 6.62607015e-34, k = 1.380649e-23;

  // Planck blackbody spectral radiance in microflicks per µm
  function planckMicroflickPerUm(lambda_um, T){
    const lambda_m = lambda_um * 1e-6;
    const exponent = (h*c)/(lambda_m*k*T);
    const denom = Math.expm1(Math.min(exponent, 700));
    const B_per_m = (2*h*c*c)/(lambda_m**5 * denom);
    const B_per_um = B_per_m * 1e-6; // per micron
    return B_per_um * 1e-4 ; // flicks
  }

  function draw(tempK){
    const lambdaPeakUm = 2898 / tempK; // Wien's displacement (µm·K)
    const xMax = Math.min(15, Math.max(1.5, lambdaPeakUm * 6));
    const domain = [0.2, xMax];

    const data = Array.from({length: samples}, (_,i)=>{
      const t = i/(samples-1);
      const l = domain[0]*(1-t)+domain[1]*t;
      return {lambda: l, B: planckMicroflickPerUm(l, tempK)};
    });

    const band = [{x1: 0.3, x2: 0.8, y1: 0, y2: Math.max(...data.map(d=>d.B))}];

    const plot = Plot.plot({
      height: 340,
      marginLeft: 72,
      marginBottom: 44,
      x: {label: "Wavelength (µm)", domain},
      y: {label: "Spectral radiance (flicks)", grid: true},
      color: {legend: false},
      marks: [
        // Visible band overlay (0.3–0.8 µm) drawn as background
        Plot.rectY(band, {x1: "x1", x2: "x2", y1: "y1", y2: "y2", fill: "url(#vis-spectrum)", opacity: 1}),
        // Curve on top of the overlay
        Plot.line(data, {x: "lambda", y: "B", stroke: "#8b2f1c"}),
        // Optional edge accents for the visible window
        Plot.ruleX([band[0].x1, band[0].x2], {stroke: "#000", strokeOpacity: 0.25}),
        Plot.ruleY([0])
      ]
    });

    // Define the gradient used by the overlay
    const svg = plot; // the root <svg>
    const defs = svg.querySelector('defs') || svg.insertBefore(document.createElementNS('http://www.w3.org/2000/svg','defs'), svg.firstChild);
    let lg = defs.querySelector('#vis-spectrum');
    if (!lg){
      lg = document.createElementNS('http://www.w3.org/2000/svg','linearGradient');
      lg.setAttribute('id','vis-spectrum');
      lg.setAttribute('x1','0%'); lg.setAttribute('x2','100%');
      lg.setAttribute('y1','0%'); lg.setAttribute('y2','0%');
      const stops = [
        [0.00,'#2e00ff',0], [0.02,'#2e00ff',0.85], [0.10,'#3f00ff',0.95],
        [0.18,'#0066ff',0.98], [0.30,'#00d5ff',0.98], [0.44,'#00ff3b',0.98],
        [0.58,'#ffff00',0.98], [0.72,'#ff7f00',0.98], [0.86,'#ff0000',0.95],
        [0.96,'#ff0000',0.80], [1.00,'#ff0000',0]
      ];
      for (const [o,cx,op] of stops){
        const s = document.createElementNS('http://www.w3.org/2000/svg','stop');
        s.setAttribute('offset',String(o));
        s.setAttribute('stop-color',cx);
        s.setAttribute('stop-opacity',String(op));
        lg.appendChild(s);
      }
      defs.appendChild(lg);
    }

    return plot;
  }

  // Initial render
  let currentTemp = 300;
  let svg = draw(currentTemp);
  container.append(svg);

  // Controls below the graph: slider + live value
  const controls = document.createElement("div");
  controls.style.display = "flex";
  controls.style.alignItems = "center";
  controls.style.gap = "0.5rem";
  controls.style.marginTop = "0.75rem";

  const slider = document.createElement("input");
  slider.type = "range";
  slider.min = "240";
  slider.max = "5000";
  slider.step = "1";
  slider.value = String(currentTemp);
  slider.style.flex = "1 1 auto";

  const valueWrap = document.createElement("div");
  valueWrap.style.display = "flex";
  valueWrap.style.flexDirection = "column";
  valueWrap.style.alignItems = "flex-end";

  const valueEl = document.createElement("span");
  valueEl.textContent = `${currentTemp} K`;
  valueEl.style.minWidth = "4.5rem";
  valueEl.style.textAlign = "right";
  valueEl.style.color = "var(--theme-foreground)";
  valueEl.style.fontWeight = "600";
  valueEl.style.fontVariantNumeric = "tabular-nums";

  const valueElC = document.createElement("span");
  valueElC.textContent = `${(currentTemp - 273.15).toFixed(1)} °C`;
  valueElC.style.fontSize = "0.85rem";
  valueElC.style.color = "var(--theme-foreground-muted)";
  valueElC.style.fontVariantNumeric = "tabular-nums";

  valueWrap.append(valueEl, valueElC);

  controls.append(slider, valueWrap);
  container.append(controls);

  slider.addEventListener('input', () => {
    currentTemp = Number(slider.value);
    valueEl.textContent = `${currentTemp} K`;
    valueElC.textContent = `${(currentTemp - 273.15).toFixed(1)} °C`;
    const next = draw(currentTemp);
    svg.replaceWith(next);
    svg = next;
  });

  return container;
})()
```

  </div>
  </div>
  </section>

  <section class="alg-section alg-narrow">
    <h3>Wien’s approximation (derivation)</h3>
    <p>Start from Planck’s law and let ${tex`x = \tfrac{hc}{k_B T\,\lambda}`}. Then</p>
    <p>${tex.block`B_\lambda(T) = \frac{2hc^2}{\lambda^5}\,\frac{1}{e^{x}-1}`}</p>
    <p>Multiply numerator and denominator by ${tex`e^{-x}`} to expose a small quantity in the Wien regime:</p>
    <p>${tex.block`\frac{1}{e^{x}-1} = \frac{e^{-x}}{1-e^{-x}}`}</p>
    <p>As ${tex`x`} grows (short ${tex`\lambda`} or high ${tex`T`}), ${tex`e^{-x}`} decays rapidly and ${tex`1-e^{-x}`} approaches 1.</p>

  <div class="alg-figure" style="max-width:720px;margin:0.5rem auto 0">

```js
(() => {
  const xs = Array.from({length: 400}, (_, i) => i * 0.04); // 0..6
  const data = xs.map(x => ({x, y: 1 - Math.exp(-x)}));
  return Plot.plot({
    height: 220,
    marginLeft: 72,
    marginBottom: 44,
    x: {label: "x", domain: [0, 15]},
    y: {label: "1 - e^{-x}", domain: [0, 1], grid: true},
    color: {legend: false},
    marks: [
      Plot.ruleY([1], {stroke: "#000", strokeOpacity: 0.25}),
      Plot.line(data, {x: "x", y: "y", stroke: "#8b2f1c", strokeWidth: 2})
    ]
  });
})()
```
  <p style="text-align:center;color:var(--theme-foreground-muted);margin:0.35rem 0 0">${tex`1-e^{-x}`} quickly approaches 1 (e.g., ${tex`x\ge 3`} gives ${tex`1-e^{-x}>0.95`}).</p>
    </div>

  <p>Hence ${tex`\tfrac{e^{-x}}{1-e^{-x}} \approx e^{-x}`} and we obtain</p>
  <p>${tex.block`\frac{e^{-x}}{1-e^{-x}} \approx e^{-x} \qquad (x\gg 1)`}</p>
  <p>Substituting back gives Wien’s approximation:</p>
  <p>${tex.block`B_\lambda(T)\;\approx\;\frac{2hc^2}{\lambda^5}\,\exp\!\left(-\tfrac{hc}{k_B T\,\lambda}\right)`}</p>
  <div class="alg-figure" style="max-width:720px;margin:0.75rem auto 0">
   
```js
(() => {
  const container = document.createElement("div");
  const c = 299792458, h = 6.62607015e-34, k = 1.380649e-23;
  function planckFlick(lambda_um, T){
    const lambda_m = lambda_um * 1e-6;
    const exponent = (h*c)/(lambda_m*k*T);
    const denom = Math.exp(Math.min(exponent, 700)) - 1;
    const B_per_m = (2*h*c*c)/((lambda_m**5) * denom);
    const B_per_um = B_per_m * 1e-6; return B_per_um * 1e-4;
  }
  function wienFlick(lambda_um, T){
    const lambda_m = lambda_um * 1e-6;
    const exponent = (h*c)/(lambda_m*k*T);
    const B_per_m = (2*h*c*c)/(lambda_m**5) * Math.exp(-Math.min(exponent, 700));
    const B_per_um = B_per_m * 1e-6; return B_per_um * 1e-4;
  }

  const domain = [0.2, 40];
  const samples = 300;
  const lambdas = Array.from({length: samples}, (_, i) => domain[0] + (domain[1]-domain[0]) * (i/(samples-1)));

  function ensureVisGradient(svg){
    const defs = svg.querySelector('defs') || svg.insertBefore(document.createElementNS('http://www.w3.org/2000/svg','defs'), svg.firstChild);
    let lg = defs.querySelector('#vis-spectrum');
    if (!lg){
      lg = document.createElementNS('http://www.w3.org/2000/svg','linearGradient');
      lg.setAttribute('id','vis-spectrum');
      lg.setAttribute('x1','0%'); lg.setAttribute('x2','100%');
      lg.setAttribute('y1','0%'); lg.setAttribute('y2','0%');
      const stops = [
        [0.00,'#2e00ff',0],[0.02,'#2e00ff',0.85],[0.10,'#3f00ff',0.95],
        [0.18,'#0066ff',0.98],[0.30,'#00d5ff',0.98],[0.44,'#00ff3b',0.98],
        [0.58,'#ffff00',0.98],[0.72,'#ff7f00',0.98],[0.86,'#ff0000',0.95],
        [0.96,'#ff0000',0.80],[1.00,'#ff0000',0]
      ];
      for (const [o,cx,op] of stops){
        const s = document.createElementNS('http://www.w3.org/2000/svg','stop');
        s.setAttribute('offset',String(o)); s.setAttribute('stop-color',cx); s.setAttribute('stop-opacity',String(op));
        lg.appendChild(s);
      }
      defs.appendChild(lg);
    }
  }

  function draw(tempK){
    const planck = lambdas.map(lambda => ({lambda, B: planckFlick(lambda, tempK)}));
    const wien = lambdas.map(lambda => ({lambda, B: wienFlick(lambda, tempK)}));
    const ymax = Math.max(...planck.map(d=>d.B), ...wien.map(d=>d.B));
    const band = [{x1: 0.3, x2: 0.8, y1: 0, y2: ymax}];
    const plotMain = Plot.plot({
      height: 240,
      marginLeft: 72,
      marginBottom: 44,
      x: {label: "Wavelength (µm)", domain},
      y: {label: "Spectral radiance (flicks)", grid: true},
      color: {legend: false},
      marks: [
        Plot.rectY(band, {x1: "x1", x2: "x2", y1: "y1", y2: "y2", fill: "url(#vis-spectrum)", opacity: 1}),
        // Planck (dashed)
        Plot.line(planck, {x: "lambda", y: "B", stroke: "#8b2f1c", strokeDasharray: "4,2"}),
        // Wien (solid)
        Plot.line(wien, {x: "lambda", y: "B", stroke: "#4e79a7"}),
        Plot.ruleY([0])
      ]
    });
    ensureVisGradient(plotMain);

    // Legend for the main plot (between main and difference plots)
    function legendItem(label, stroke, dash){
      const item = document.createElement("div");
      item.style.display = "inline-flex";
      item.style.alignItems = "center";
      item.style.gap = "0.4rem";
      const s = document.createElementNS('http://www.w3.org/2000/svg','svg');
      s.setAttribute('width','28'); s.setAttribute('height','10'); s.setAttribute('viewBox','0 0 28 10');
      const l = document.createElementNS('http://www.w3.org/2000/svg','line');
      l.setAttribute('x1','0'); l.setAttribute('y1','5'); l.setAttribute('x2','28'); l.setAttribute('y2','5');
      l.setAttribute('stroke', stroke); l.setAttribute('stroke-width','2');
      if (dash) l.setAttribute('stroke-dasharray', dash);
      s.appendChild(l);
      const t = document.createElement('span');
      t.textContent = label; t.style.color = "var(--theme-foreground-muted)"; t.style.fontSize = "12px";
      item.append(s, t);
      return item;
    }
    const legend = document.createElement("div");
    legend.style.display = "flex";
    legend.style.alignItems = "center";
    legend.style.gap = "1rem";
    legend.style.marginTop = "0.25rem";
    legend.append(
      legendItem("Planck", "#8b2f1c", "4,2"),
      legendItem("Wien", "#4e79a7", "")
    );

    // Difference plot (Planck - Wien)
    // Relative difference in percent: (Planck - Wien) / Planck * 100
    const diff = lambdas.map((lambda, i) => {
      const denom = Math.max(planck[i].B, 1e-30);
      return {lambda, D: (planck[i].B - wien[i].B) / denom * 100};
    });
    // Fixed percent axis 0..100
    const bandDiff = [{x1: 0.3, x2: 0.8, y1: 0, y2: 100}];
    const plotDiff = Plot.plot({
      height: 200,
      marginLeft: 72,
      marginTop: 25,
      marginBottom: 44,
      x: {label: "Wavelength (µm)", domain},
      y: {label: "Relative difference (%)", grid: true, domain: [0, 100]},
      marks: [
        Plot.rectY(bandDiff, {x1: "x1", x2: "x2", y1: "y1", y2: "y2", fill: "url(#vis-spectrum)", opacity: 1}),
        Plot.line(diff, {x: "lambda", y: "D", stroke: "#555", strokeWidth: 1.75}),
        Plot.ruleY([0])
      ]
    });
    ensureVisGradient(plotDiff);

    const wrap = document.createElement("div");
    const caption = document.createElement("div");
    caption.textContent = "Now, let's look at the percentage difference between Planck and Wien.";
    caption.style.textAlign = "left";
    caption.style.color = "black";
    caption.style.fontSize = "17px / 1.5 var(--serif);";
    caption.style.margin = "2rem 0rem 2rem 0rem";
    wrap.append(plotMain, legend, caption, plotDiff);
    return wrap;
  }

  let currentTemp = 1500;
  let svg = draw(currentTemp);
  container.append(svg);

  // legend moved inside draw() above the difference plot

  // slider controls
  const controls = document.createElement("div");
  controls.style.display = "flex";
  controls.style.alignItems = "center";
  controls.style.gap = "0.5rem";
  controls.style.marginTop = "0.5rem";
  const slider = document.createElement("input");
  slider.type = "range"; slider.min = "240"; slider.max = "5000"; slider.step = "1"; slider.value = String(currentTemp);
  slider.style.flex = "1 1 auto";
  const valueWrap = document.createElement("div");
  valueWrap.style.display = "flex"; valueWrap.style.flexDirection = "column"; valueWrap.style.alignItems = "flex-end";
  const valueK = document.createElement("span");
  valueK.textContent = `${currentTemp} K`;
  valueK.style.minWidth = "4.5rem"; valueK.style.textAlign = "right"; valueK.style.color = "var(--theme-foreground)"; valueK.style.fontWeight = "600"; valueK.style.fontVariantNumeric = "tabular-nums";
  const valueC = document.createElement("span");
  valueC.textContent = `${(currentTemp - 273.15).toFixed(1)} °C`;
  valueC.style.fontSize = "0.85rem"; valueC.style.color = "var(--theme-foreground-muted)"; valueC.style.fontVariantNumeric = "tabular-nums";
  valueWrap.append(valueK, valueC);
  controls.append(slider, valueWrap);
  container.append(controls);

  slider.addEventListener('input', () => {
    currentTemp = Number(slider.value);
    valueK.textContent = `${currentTemp} K`;
    valueC.textContent = `${(currentTemp - 273.15).toFixed(1)} °C`;
    const next = draw(currentTemp); svg.replaceWith(next); svg = next;
  });

  return container;
})()
```
  </div>

  <p style="margin:1.25rem 0 0.5rem">
    Wien’s approximation is a simpler, lighter version of Planck’s law. It preserves the essential
    exponential falloff, which makes it handy for quick reasoning and for building intuition. It is
    still an approximation with clear bounds: at long wavelengths the error grows, and at very high
    temperatures it can miss details that the full Planck curve captures. As temperature drops (or as
    wavelength shortens), that gap naturally narrows.
  </p>
  <p style="margin:0 0 0.75rem">
    If we focus on the LWIR window (8–15 µm) and think in everyday Earth terms—roughly 200–400 K—
    the errors are usually only a few percent. That’s small enough to trust the overall shape and
    trends, keep the math friendly, and still say something useful about temperature and emissivity.
  </p>

  <div class="alg-figure" style="max-width:720px;margin:0.5rem auto 1.5rem">

```js
(() => {
  const c = 299792458, h = 6.62607015e-34, k = 1.380649e-23;
  function planckFlick(lambda_um, T){
    const lambda_m = lambda_um * 1e-6;
    const exponent = (h*c)/(lambda_m*k*T);
    const denom = Math.exp(Math.min(exponent, 700)) - 1;
    const B_per_m = (2*h*c*c)/((lambda_m**5) * denom);
    const B_per_um = B_per_m * 1e-6; return B_per_um * 1e-4;
  }
  function wienFlick(lambda_um, T){
    const lambda_m = lambda_um * 1e-6;
    const exponent = (h*c)/(lambda_m*k*T);
    const B_per_m = (2*h*c*c)/(lambda_m**5) * Math.exp(-Math.min(exponent, 700));
    const B_per_um = B_per_m * 1e-6; return B_per_um * 1e-4;
  }

  const lwirDomain = [8, 15];
  const tempK = 300; // representative LWIR earth-surface temperature
  const samples = 240;
  const lambdas = Array.from({length: samples}, (_, i) => lwirDomain[0] + (lwirDomain[1]-lwirDomain[0]) * (i/(samples-1)));
  const planck = lambdas.map(lambda => ({lambda, B: planckFlick(lambda, tempK)}));
  const wien = lambdas.map(lambda => ({lambda, B: wienFlick(lambda, tempK)}));
  const diff = lambdas.map((lambda, i) => {
    const denom = Math.max(planck[i].B, 1e-30);
    return {lambda, D: (planck[i].B - wien[i].B) / denom * 100};
  });

  return Plot.plot({
    height: 200,
    marginLeft: 96,
    marginTop: 25,
    marginBottom: 44,
    x: {label: "Wavelength (µm)", domain: lwirDomain},
    y: {label: "Relative difference (%)", grid: true, domain: [0, 5]},
    marks: [
      Plot.line(diff, {x: "lambda", y: "D", stroke: "#555", strokeWidth: 1.75}),
      // Temperature note inside the plot area
      Plot.text([{x: lwirDomain[1] - 0.3, y: 4.5, label: `T = ${tempK} K`}], {x: "x", y: "y", text: "label", textAnchor: "end", dx: -4, dy: -4, fill: "#555", fontSize: 12}),
      Plot.ruleY([0])
    ]
  });
})()
```
  </div>

  <p style="margin:1rem 0 0.75rem">
    A brief note on units and geometry: radiance measures power per area, per wavelength, per steradian
    (direction). Exitance (also called emissive power) integrates over all directions in the hemisphere
    above the surface. For an ideal, Lambertian blackbody the two are linked by a simple factor of
    ${tex`\pi`}:
  </p>
  <p>${tex.block`M_\lambda(T)\;=\;\pi\,B_\lambda(T)\;=\;\frac{2\pi h c^2}{\lambda^5}\,\frac{1}{\exp\!\left(\tfrac{hc}{\lambda k T}\right)-1}`}</p>


  <p style="margin:0.25rem 0 0.5rem">
    Up to this point we’ve looked at radiance. If we are instead interested in exitance, we include
    a ${tex`\pi`} factor. For Wien’s approximation that simply becomes:
  </p>
  <p>${tex.block`M_\lambda^{\mathrm{Wien}}(T)\;\approx\;\frac{2{\color{#8b2f1c}\pi}hc^2}{\lambda^5}\,\exp\!\left(-\tfrac{hc}{\lambda k T}\right)`}</p>

  <h4 style="margin:1rem 0 0.5rem">From exitance with Wien to a linear form</h4>
  <p>Start from a band‑wise measured signal ${tex`R_i`} and model it as emissivity times the
    blackbody exitance under Wien’s approximation:</p>
  <p>${tex.block`R_i\;\approx\;\varepsilon_i\,M_\lambda^{\mathrm{Wien}}(\lambda_i,T)\;=\;\varepsilon_i\,\frac{2\pi h c^2}{\lambda_i^5}\,\exp\!\left(-\tfrac{hc}{\lambda_i k T}\right)`}</p>
  <p>Introduce constants ${tex`C_1=2\pi h c^2`} and ${tex`C_2=\tfrac{hc}{k_B}`}. We are interested in
    radiance rather than exitance, so we divide by ${tex`\pi`}:</p>
  <p>${tex.block`R_i\;\approx\;\varepsilon_i\,\frac{C_1/\pi}{\lambda_i^5}\,\exp\!\left(-\tfrac{C_2}{\lambda_i T}\right)`}</p>
  <p>Take logarithms:</p>
  <p>${tex.block`\ln R_i\;\approx\;\ln\varepsilon_i\; +\; \ln C_1\; -\;\ln\pi\; -\;5\,\ln\lambda_i\; -\; \tfrac{C_2}{\lambda_i T}`}</p>
  <p>Multiply by ${tex`\lambda_i`} and isolate ${tex`C_2/T`}:</p>
  <p>${tex.block`\tfrac{C_2}{T}\;\approx\;-\,\lambda_i\ln R_i\; +\; \lambda_i\ln\varepsilon_i\; +\; \lambda_i\ln C_1\; -\; \lambda_i\ln\pi\; -\;5\,\lambda_i\ln\lambda_i`}</p>
  <h4 style="margin:0.75rem 0 0.35rem">Residualization (step by step)</h4>
  <p style="margin:0 0 0.35rem">We’ll subtract the band‑average identity from the per‑band identity. First, write them side‑by‑side:</p>
  <div class="eq-balance">${tex.block`
  \tfrac{C_2}{T} \;\approx\; -\,\lambda_i\ln R_i\; +\; \lambda_i\ln\varepsilon_i\; +\; \lambda_i\ln C_1\; -\; \lambda_i\ln\pi\; -\;5\,\lambda_i\ln\lambda_i
  `}</div>
  <div class="eq-balance">${tex.block`
  \underbrace{\frac{1}{n}\sum_{j=1}^n\tfrac{C_2}{T}}_{\text{band avg}} \;\approx\; -\;\frac{1}{n}\sum_{j=1}^n \lambda_j\ln R_j\; +\; \frac{1}{n}\sum_{j=1}^n \lambda_j\ln\varepsilon_j\; +\;\Big(\ln C_1 - \ln\pi\Big)\,\frac{1}{n}\sum_{j=1}^n \lambda_j\; -\;5\,\frac{1}{n}\sum_{j=1}^n \lambda_j\ln\lambda_j
  `}</div>
  <p style="margin:0 0 0.35rem">Now subtract the second line from the first (left from left, right from right). Because ${tex`\tfrac{C_2}{T}`} is constant across
    wavelength, the left‑hand side cancels to zero:</p>
  <div class="eq-balance">${tex.block`
  0\;\approx\;\color{#4e79a7}{\Big(-\,\lambda_i\ln R_i\;+\;\frac{1}{n}\sum_{j} \lambda_j\ln R_j}\Big)
  \;\color{#000}{+}\;\color{#8b2f1c}{\Big(\lambda_i\ln\varepsilon_i}\;\color{#000}{-}\;\color{#8b2f1c}{\frac{1}{n}\sum_{j} \lambda_j\ln\varepsilon_j}\Big)\color{#4e79a7}
  \;+\;\Big(\lambda_i\ln C_1 - \lambda_i\ln\pi - (\ln C_1 - \ln\pi)\,\frac{1}{n}\sum_{j}\lambda_j\Big)
  \;-\;5\Big(\lambda_i\ln\lambda_i - \frac{1}{n}\sum_{j}\lambda_j\ln\lambda_j\Big)
  `}</div>
  <p style="margin:0 0 0.35rem">keep the radiance block on the right and move the emissivity to the left; this exposes a structure:</p>
  <div class="eq-balance">${tex.block`
  \color{#8b2f1c}{\lambda_i\ln\varepsilon_i - \frac{1}{n}\sum_{j} \lambda_j\ln\varepsilon_j}
  \\ \qquad=\; \color{#4e79a7}{\lambda_i\ln R_i - \frac{1}{n}\sum_{j} \lambda_j\ln R_j}
  \;+\;5\Big(\lambda_i\ln\lambda_i - \frac{1}{n}\sum_{j}\lambda_j\ln\lambda_j\Big)
  \;-\;\Big(\ln C_1 - \ln\pi\Big)\Big(\lambda_i - \frac{1}{n}\sum_{j}\lambda_j\Big)
  `}</div>
  <p style="margin:0 0 0.35rem">Group repeated patterns and define auxiliary quantities to simplify notation:</p>
  <div class="eq-balance">${tex.block`
   \mu_\alpha\;:=\;\frac{1}{n}\sum_{j=1}^n \lambda_j\ln\varepsilon_j\;\;\;\;\;\;\bar{\lambda}\;:=\;\frac{1}{n}\sum_{j=1}^n \lambda_j
  `}</div>
  <div class="eq-balance">${tex.block`
   \kappa_i\;:=\;5\Big(\lambda_i\ln\lambda_i - \frac{1}{n}\sum_{j}\lambda_j\ln\lambda_j\Big)\; -\;\Big(\ln C_1 - \ln\pi\Big)\Big(\lambda_i - \bar{\lambda}\Big)
  `}</div>
  <p style="margin:0 0 0.35rem">With these, the equation becomes a compact per‑band relation:</p>
  <div class="eq-balance numbered">${tex.block`
   \underbrace{\lambda_i\,\ln\varepsilon_i}_{\text{\small unknown}}\; -\; \underbrace{\mu_\alpha}_{\text{\small needs to be estimated}}\;=
   \;\underbrace{\Big(\lambda_i\,\ln R_i - \tfrac{1}{n}\sum_{j}\lambda_j\,\ln R_j\Big)}_{\text{\small data}}\; +\; \underbrace{\kappa_i}_{\text{\small constant}} \tag{1}
  `}</div>

  <h4 style="margin:0.9rem 0 0.35rem">Defining \(\alpha_i\) and estimating the mean</h4>
  <p>Define the centered emissivity term</p>
  <div class="eq-balance">${tex.block`\alpha_i\;:=\;\lambda_i\,\ln\varepsilon_i\; -\;\mu_\alpha`}</div>
  <p>and let its sample variance for a spectrum be</p>
  <div class="eq-balance">${tex.block`\nu_\alpha\;:=\;\frac{1}{n-1}\sum_{j=1}^{n}\alpha_j^2`}</div>
  <p>In practice, the mean term can be estimated by a simple regression that links the magnitude of
     the deviations (via ${tex`\nu_\alpha`}) to the offset ${tex`\mu_\alpha`} across spectra:</p>
  <div class="eq-balance numbered">${tex.block`\mu_\alpha\;\approx\;c\;\nu_\alpha^{\,1/\chi} \tag{2}`}</div>
  <p >Here ${tex`c`} and ${tex`\chi`} are fitted constants determined from laboratory calibration.</p>
  <p>With an estimate of ${tex`\mu_\alpha`} from Eq.&nbsp;(2) and the identity in Eq.&nbsp;(1), we can recover emissivity per band:</p>
  <div class="eq-balance">${tex.block`\varepsilon_i\;=\;\exp\!\left(\frac{\mu_\alpha\; +\; \kappa_i\; +\; \Big(\lambda_i\,\ln R_i\; -\; \tfrac{1}{n}\sum_{j}\lambda_j\,\ln R_j\Big)}{\lambda_i}\right)`}</div>
  </section>

  <section class="alg-section alg-refs">
    <h3>References</h3>
    <ol>
      <li>Kealy, Peter S. "Estimation of emissivity and temperature using alpha coefficients." Proceedings of the Second Thermal Infrared Multispectral Scanner (TIMS) Workshop. 1990.</li>
      <li>Gillespie, Alan R., et al. "Temperature/emissivity separation algorithm theoretical basis document, version 2.4." ATBD contract NAS5-31372, NASA (1999).</li>

    </ol>
  </section>
</div>
