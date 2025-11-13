---
toc: false
---

<link rel="stylesheet" href="/algorithms/algorithm.css">

<div>
  <a href="/" class="alg-back" aria-label="Back to home">←</a>
</div>

<div class="alg-container">
  <header class="alg-hero">
    <h1>NEM</h1>
    <p>Normalized Emissivity Method</p>
  </header>

  <section class="alg-meta">
    <div><strong>Authors</strong>: Gillespie, A. R.</div>
    <span class="sep"></span>
    <div><strong>Date</strong>: 1985</div>
  </section>

  <section class="alg-section alg-narrow">
  

<p style="margin-top:0.8rem;">
  In NEM, the pixel with the <strong>maximum radiance</strong> is assumed to have the
  <strong>maximum emissivity</strong> \(\varepsilon_{\max}\).
  This value is not taken as 1.0 in practice: real surfaces are not perfect blackbodies.
  
</p>

<p style="margin-top:0.6rem;">
  For example, in ASTER assumes 
  \(\varepsilon_{\max} \approx 0.99\), which reflects the fact that most natural
  surfaces (vegetation, water, moist soil) exhibit emissivities between 0.97 and 0.995
  in the 8–12&nbsp;µm thermal region.
</p>

<p style="margin-top:0.6rem;">
  When NEM is applied to other thermal sensors, similar values are used.
  For MODIS, published TES studies typically assume 
  \(\varepsilon_{\max} = 0.97\text{–}0.99\).
  For Landsat TIR, implementations generally adopt 
  \(\varepsilon_{\max} = 0.985\text{–}0.99\).
</p>






  From this reference band, the <strong>brightness temperature</strong> is estimated using the inverse of Planck’s law, which serves as the basis for deriving the kinetic temperature and emissivity of the remaining pixels.
</p>
  

  </br>

  
  <h3 style="margin:1rem 0 0.5rem">Brightness Temperature</h3>
  <p>

<p>
  The <strong>brightness temperature</strong> (<strong>T<sub>b</sub></strong>) is obtained by
  inverting Planck’s law using the measured spectral radiance (<strong>L<sub>λ</sub></strong>).
</p>

<!-- Planck's Law  -->
<p style="margin-top:0.6rem;">
  <strong>Planck’s law:</strong>
</p>
<p style="margin:0.35rem 0;">
  \[
    B_\lambda(T) \;=\; \frac{2hc^2}{\lambda^5}\,\frac{1}{\exp\!\Big(\frac{hc}{\lambda k T}\Big)-1}
    \;=\; \frac{c_1}{\lambda^5}\,\frac{1}{\exp\!\Big(\frac{c_2}{\lambda T}\Big)-1},
  \]
  
</p>
<p style="text-align:center;  margin-top:0;">
  where \(c_1 = 2hc^2\) and \(c_2 = \dfrac{hc}{k}\)
</p>

<!-- Definition of Tb -->
<p style="margin-top:0.8rem;">
  By definition, the brightness temperature \(T_b\) satisfies \(L_\lambda = B_\lambda(T_b)\). Substituting:
</p>
<p style="margin:0.35rem 0;">
  \[
    L_\lambda \;=\; \frac{c_1}{\lambda^5}\,\frac{1}{\exp\!\Big(\frac{c_2}{\lambda T_b}\Big)-1}
  \]
</p>

<!-- Step-by-step derivation -->
<p style="margin-top:0.8rem;"><strong>Step-by-step derivation:</strong></p>

<ol style="padding-left:1.2rem; font-size:0.95rem;">
  <li style="margin:0.35rem 0;">
    Multiply both sides by the denominator:
    \[
      L_\lambda\Big(\exp\!\Big(\frac{c_2}{\lambda T_b}\Big)-1\Big) \;=\; \frac{c_1}{\lambda^5}
    \]
  </li>
  <li style="margin:0.35rem 0;">
    Divide by \(L_\lambda\):
    \[
      \exp\!\Big(\frac{c_2}{\lambda T_b}\Big) - 1 \;=\; \frac{c_1}{\lambda^5 L_\lambda}
    \]
  </li>
  <li style="margin:0.35rem 0;">
    Add 1 to both sides:
    \[
      \exp\!\Big(\frac{c_2}{\lambda T_b}\Big) \;=\; 1 + \frac{c_1}{\lambda^5 L_\lambda}
    \]
  </li>
  <li style="margin:0.35rem 0;">
    Take the natural logarithm:
    \[
      \frac{c_2}{\lambda T_b} \;=\; \ln\!\Bigg(1 + \frac{c_1}{\lambda^5 L_\lambda}\Bigg)
    \]
  </li>
  <li style="margin:0.35rem 0;">
    Solve for \(T_b\):
    <div style="text-align:center; margin-top:0.35rem;font-size:1.2rem;">
      \[
        \boxed{
          T_b \;=\; \frac{c_2}{\lambda \,\ln\!\left(1 + \dfrac{c_1}{\lambda^5 L_\lambda}\right)}
        }
      \]
    </div>
  </li>
</ol>


</p>



   
  

<!-- Contenedor donde irá el gráfico -->
<div id="nem-plot" class="alg-figure bleed" style="margin-top:1rem;"></div>

<!-- Controles interactivos -->
<div style="margin-top:2rem; text-align:center;">
  <label for="tempRange" style="font-weight:600; font-size:0.95rem;">Surface Temperature (K):</label><br>
  <input 
    type="range" 
    id="tempRange" 
    min="500" 
    max="5000" 
    value="330" 
    step="1"
    style="width:350px; accent-color:#ff0000; vertical-align:middle;">
  <div id="tempValue" style="font-weight:700; font-size:1rem; margin-top:0.4rem;">330 K</div>

  <div style="margin-top:1rem;">
    <label for="epsRange" style="font-weight:600; font-size:0.95rem;">Emissivity (ε):</label><br>
    <input
      type="range"
      id="epsRange"
      min="0.30"
      max="1.00"
      value="0.95"
      step="0.01"
      style="width:350px; accent-color:#ff0000; vertical-align:middle;">
    <div id="epsValue" style="font-weight:700; font-size:1rem; margin-top:0.4rem;">ε = 0.95</div>
  </div>
</div>

<script type="module">
  import * as Plot from "https://cdn.jsdelivr.net/npm/@observablehq/plot@0.6/+esm";

  // --- Physical constants (SI) ---
  const c = 299792458;       // m/s
  const h = 6.62607015e-34;  // J·s
  const k = 1.380649e-23;    // J/K
  const c1 = 2 * h * c * c;  // 2hc^2
  const c2 = (h * c) / k;    // hc/k

  const lambda_um = 10;
  const lambda_m = lambda_um * 1e-6;
  const samples = 300;
  const Tmin = 500, Tmax = 5000;

  // Conversion factor → microflicks (μf)
  // 1 W/m²·sr·μm = 10^6 μf
  const toMicroflicks = (L_SI) => L_SI * 1e6;

  // --- Helpers ---
  function planckRadiance_SI(T, λ) {
    const expo = (h * c) / (λ * k * T);
    return (c1 / Math.pow(λ, 5)) / (Math.exp(expo) - 1);
  }

  function Tb_from_Lμf(L_μf, λ) {
    // Convert back from μf to W/m²·sr·μm
    const L_SI = L_μf / 1e6;
    const λ5 = Math.pow(λ, 5);
    const lnTerm = Math.log(1 + c1 / (λ5 * L_SI));
    return c2 / (λ * lnTerm);
  }

  // --- Data ---
  function buildCurve() {
    const Lmin = planckRadiance_SI(Tmin, lambda_m);
    const Lmax = planckRadiance_SI(Tmax, lambda_m) * 1.05;
    const data = [];
    for (let i = 0; i < samples; i++) {
      const t = i / (samples - 1);
      const L_SI = Lmin * Math.pow(Lmax / Lmin, t);
      const L_μf = toMicroflicks(L_SI);
      const Tb = Tb_from_Lμf(L_μf, lambda_m);
      data.push({ L_μf, Tb });
    }
    return data;
  }

  function marker(T, eps) {
    const L_SI = eps * planckRadiance_SI(T, lambda_m);
    const L_μf = toMicroflicks(L_SI);
    const Tb = Tb_from_Lμf(L_μf, lambda_m);
    return { L_μf, Tb, T, eps };
  }

  // --- Tooltip HTML element ---
  const tooltip = document.createElement("div");
  tooltip.style.position = "absolute";
  tooltip.style.pointerEvents = "none";
  tooltip.style.padding = "6px 8px";
  tooltip.style.background = "rgba(0,0,0,0.75)";
  tooltip.style.color = "#fff";
  tooltip.style.borderRadius = "6px";
  tooltip.style.fontSize = "0.85rem";
  tooltip.style.display = "none";
  tooltip.style.zIndex = "1000";
  document.body.appendChild(tooltip);

  // --- Draw plot ---
  function decadeTicks(min, max) {
  const e0 = Math.floor(Math.log10(min));
  const e1 = Math.ceil(Math.log10(max));
  const ticks = [];
  for (let e = e0; e <= e1; e++) ticks.push(10 ** e);
  return ticks;
}

// --- Draw plot (pretty x-axis in μf) ---
function renderPlot(T, eps) {
  const data = buildCurve();
  const m = marker(T, eps);

  // límites X (en μf) y formato
  const Lmin = Math.max(1e-6, data[0].L_μf);
  const Lmax = data[data.length - 1].L_μf;

  const chart = Plot.plot({
    height: 350,
    marginLeft: 70,
    marginBottom: 54,
    x: {
      label: "Measured Radiance L (μf)",
      domain: [Lmin, Lmax],
      grid: true,
      tickFormat: d => d.toExponential(1) // mantiene legible sin saturar
    },
    y: { 
      label: "Brightness Temperature T_b (K)", 
      domain: [500, 5500],   // <-- límite inferior 500 K como pediste
      grid: true 
    },
    marks: [
      Plot.line(data, { x: "L_μf", y: "Tb", stroke: "#f30606ff", strokeWidth: 2 }),
      Plot.ruleX([m.L_μf], { stroke: "#888", strokeDasharray: "4 4" }),
      Plot.ruleY([m.Tb],   { stroke: "#888", strokeDasharray: "4 4" }),
      Plot.dot([m],        { x: "L_μf", y: "Tb", r: 6, fill: "#000", opacity: 0.9 })
    ]
  });

  const div = document.getElementById("nem-plot");
  div.innerHTML = "";
  div.append(chart);

  // === Tooltips sobre el punto ===
  const dot = div.querySelector("circle");
  if (dot) {
    dot.addEventListener("mouseenter", () => {
      tooltip.style.display = "block";
      tooltip.textContent =
        `ε = ${eps.toFixed(2)} | ` +
        `Tₛ = ${T} K | ` +
        `L = ${m.L_μf.toExponential(2)} μf | ` +
        `T_b = ${m.Tb.toFixed(2)} K`;
    });
    dot.addEventListener("mousemove", (e) => {
      tooltip.style.left = (e.pageX + 12) + "px";
      tooltip.style.top  = (e.pageY - 10) + "px";
    });
    dot.addEventListener("mouseleave", () => {
      tooltip.style.display = "none";
    });
  }

  // Oculta el tooltip si el cursor sale del área del gráfico
  chart.addEventListener("mouseleave", () => {
    tooltip.style.display = "none";
  });
}



  // --- Sliders ---
  const tempSlider = document.getElementById("tempRange");
  const epsSlider = document.getElementById("epsRange");
  const tempValue = document.getElementById("tempValue");
  const epsValue = document.getElementById("epsValue");

  function updatePlot() {
    const T = parseInt(tempSlider.value, 10);
    const eps = parseFloat(epsSlider.value);
    tempValue.textContent = `${T} K`;
    epsValue.textContent = `ε = ${eps.toFixed(2)}`;
    renderPlot(T, eps);
  }

  tempSlider.addEventListener("input", updatePlot);
  epsSlider.addEventListener("input", updatePlot);
  updatePlot();
</script>

<p style="margin-top:0.4rem;">
  In practical implementations, the NEM temperature is often defined as the
  maximum brightness temperature over all pixels (or bands) in the scene:
</p>

<p style="text-align:center; font-size:1.1rem; margin:0.3rem 0 0.8rem 0;">
  \[
    T_{\text{nem}} \;=\; \text{max}_{i}   T_{b,i}
  \]
</p>

<p style="margin-top:0.2rem;">
  where \(T_{b,i}\) is the brightness temperature retrieved for each pixel or band
  from the measured radiance using the inverse of Planck’s law.
</p>
  
  <p style="margin-top:1rem;">
  Once the NEM temperature (<strong>T<sub>nem</sub></strong>) has been identified as the highest
  brightness temperature, the next step is to estimate the spectral emissivity
  (<strong>ε<sub>i</sub></strong>) for each band or pixel.
  Using the measured spectral radiance (<strong>L<sub>i</sub></strong>), the emissivity can be derived
  from Planck’s radiance function at the NEM temperature as:
</p>

<p style="text-align:center; font-size:1.05rem; margin-top:0.6rem;">
  \[
    \varepsilon_i = \frac{L_i}{B(\lambda, T_{nem})}
  \]
</p>

<p style="margin-top:0.6rem;">
  Here, \(B(\lambda, T_{nem})\) represents the blackbody spectral radiance computed from Planck’s law
  for the wavelength \( \lambda \) and NEM temperature \( T_{nem}\).
  This normalization step ensures that emissivity values remain consistent across all thermal bands.
</p>
<p style="margin-top:1rem;">
  With both the NEM temperature (<strong>T<sub>nem</sub></strong>) and the emissivity values
  (<strong>ε<sub>i</sub></strong>) estimated, the NEM method allows the retrieval of a
  consistent temperature–emissivity set for each pixel in the scene.
  These results can then be refined iteratively or compared with other
  temperature–emissivity separation (TES) techniques to improve accuracy.
</p>

<p>
As an example, Figure B-6—taken from the original publication—illustrates the performance of the NEM algorithm in retrieving vegetation emissivity spectra with high accuracy and consistent spectral shape.
</p>

<img src="assets/NEM.png" alt="Paper original" width="1000">

<h3 style="margin-top:1.2rem;">Iterative NEM and smoothness prior</h3>

<p style="margin-top:0.4rem;">
  In its simplest form, NEM performs a single normalization step. However, it can
  also be used <strong>iteratively</strong>: starting from an initial guess for
  \(\varepsilon_{\max}\) (for example 0.99), one computes \(T_{\text{nem}}\) and a
  first set of emissivities \(\varepsilon_i^{(0)}\). A smoothness metric of the
  emissivity spectrum (such as the spectral variance or curvature across bands)
  is then evaluated, and \(\varepsilon_{\max}\) is adjusted until this metric is
  minimized.
</p>

<p style="margin-top:0.4rem;">
  The underlying assumption is that physically realistic emissivity spectra are
  <strong>smooth</strong> in wavelength, without strong oscillations from band to
  band. The preferred solution is therefore the one that produces the smoothest
  emissivity spectrum while remaining consistent with the measured radiances.
</p>

<p style="margin-top:0.4rem;">
  A simple smoothness metric often used in TES methods is the spectral variance
  of emissitivity:
</p>

<p style="text-align:center; font-size:1.05rem; margin:0.4rem 0;">
  \[
    \mathrm{Var}(\varepsilon) \;=\; 
    \frac{1}{N}\sum_{i=1}^{N}\big(\varepsilon_i - \bar{\varepsilon}\big)^2
  \]
</p>

<p style="margin-top:0.4rem;"> The plot below shows a synthetic example of how the emissivity variance changes with the assumed value of \(\varepsilon_{\max}\). In this illustrative case, the variance reaches its minimum near \(\varepsilon_{\max} = 0.99\), which aligns with typical assumptions used in TES algorithms. </p>

<div id="nem-variance-plot" class="alg-figure" style="margin-top:0.8rem;"></div>

<script type="module">
  import * as Plot from "https://cdn.jsdelivr.net/npm/@observablehq/plot@0.6/+esm";

  // Generate a smooth upward-opening parabola
  const epsMin = 0.95;
  const epsMax = 1.03;
  const N = 40;

  // True minimum at eps = 0.99
  function varianceModel(e) {
    return 50 * Math.pow(e - 0.99, 2) + 0.002;  // Clean parabola
  }

  const varianceData = Array.from({ length: N }, (_, i) => {
    const e = epsMin + (epsMax - epsMin) * (i / (N - 1));
    return { epsMax: e, var: varianceModel(e) };
  });

  const varianceChart = Plot.plot({
    height: 260,
    marginLeft: 75,
    marginBottom: 48,
    x: {
      label: "Assumed ε_max",
      domain: [epsMin, epsMax]
    },
    y: {
      label: "Spectral variance of ε",
      grid: true
    },
    marks: [
      Plot.line(varianceData, {
        x: "epsMax",
        y: "var",
        stroke: "#d62728",
        strokeWidth: 2.5
      }),
      Plot.dot(varianceData, {
        x: "epsMax",
        y: "var",
        r: 3,
        fill: "#d62728"
      }),
      // Vertical line at the minimum
      Plot.ruleX([0.99], {
        stroke: "#444",
        strokeDasharray: "4 4"
      }),
      // Label at the minimum
      Plot.text(
        [{ epsMax: 0.99, var: varianceModel(0.99), label: "minimum at 0.99" }],
        {
          x: "epsMax",
          y: "var",
          text: "label",
          dy: -10,
          dx: 4,
          fontSize: 12,
          fill: "#111",
          fontWeight: 600
        }
      )
    ]
  });

  const varDiv = document.getElementById("nem-variance-plot");
  varDiv.innerHTML = "";
  varDiv.append(varianceChart);
</script>


<h3 style="margin-top:1rem;">Advantages</h3>
<ul>
  <li>Does not require prior knowledge of surface emissivity or material type.</li>
  <li>Can be applied to sensors with a limited number of thermal bands.</li>
  <li>Computationally simple and fast to implement.</li>
  <li>Provides reasonable accuracy in homogeneous or semi-homogeneous surfaces.</li>
</ul>

<h3 style="margin-top:1rem;">Limitations</h3>
<ul>
  <li>Assumes that at least one pixel has maximum emissivity (≈1), which may not always be true.</li>
  <li>Less accurate in heterogeneous or mixed land cover areas.</li>
  <li>Sensitive to atmospheric effects and radiometric calibration errors.</li>
  <li>Provides an approximate estimation rather than an absolute emissivity value.</li>
</ul>

<h3 style="margin-top:2rem;">Illustrative Application</h3>

<p>
  To demonstrate how the Normalized Emissivity Method (NEM) works, consider a small
  synthetic example consisting of radiance measurements at 10&nbsp;µm for five pixels.
  Radiances are expressed directly in <strong>microflicks (µf)</strong>.  
  These numbers are invented for illustration but preserve the internal behavior of NEM.
</p>

<div style="text-align:center; width:100%;">
  <table style="
      width:55%; 
      border-collapse: collapse; 
      margin-left:auto; 
      margin-right:auto; 
      font-size:0.95rem;"
  >
    <thead>
      <tr style="background-color:#f3f4f6;">
        <th style="border: 1px solid #ccc; padding: 6px; text-align:center;">Pixel</th>
        <th style="border: 1px solid #ccc; padding: 6px; text-align:center;">Radiance L (µf)</th>
      </tr>
    </thead>
    <tbody>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">1</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">10</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">2</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">12</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">3</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">15</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">4</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">14</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">5</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">13</td></tr>
    </tbody>
  </table>
</div>


<p style="margin-top:0.8rem;">
  In NEM, the pixel with the <strong>maximum radiance</strong> is assumed to have the
  <strong>maximum emissivity</strong>:
</p>

<p style="text-align:center; font-size:1.1rem; margin:0.4rem 0;">
  ε<sub>max</sub> = 0.99
</p>

<p>
  In this example, Pixel 3 has the highest radiance (15&nbsp;µf), so it is used as the
  reference pixel.  
  Applying the inverse of Planck’s law to its radiance yields the so-called
  <strong>NEM temperature</strong>:
</p>

<p style="text-align:center; font-size:1.15rem; margin:0.4rem 0;">
  <strong>T<sub>nem</sub> ≈ 330&nbsp;K</strong>
</p>

<p>
  Once T<sub>nem</sub> is known, the emissivity of each pixel follows from:
</p>

<p style="text-align:center; font-size:1.2rem; margin:0.5rem 0;">
  \( \varepsilon_i = \dfrac{L_i}{B_\lambda(T_{nem})} \)
</p>

<p style="margin-top:0.5rem;">
  Using the radiance values from the table, NEM produces the following emissivities:
</p>

<div style="text-align:center; width:100%; margin: 1.2rem 0;">
  <table style="
      width:55%;
      border-collapse: collapse;
      margin-left:auto;
      margin-right:auto;
      font-size:0.95rem;
      margin-top:0.8rem;
      margin-bottom:0.8rem;
  ">
    <thead>
      <tr style="background-color:#f3f4f6;">
        <th style="border: 1px solid #ccc; padding: 6px; text-align:center;">Pixel</th>
        <th style="border: 1px solid #ccc; padding: 6px; text-align:center;">Emissivity ε</th>
      </tr>
    </thead>
    <tbody>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">1</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">0.67</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">2</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">0.80</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">3</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">0.99</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">4</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">0.93</td></tr>
      <tr><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">5</td><td style="border: 1px solid #ccc; padding: 6px; text-align:center;">0.87</td></tr>
    </tbody>
  </table>
</div>


<p style="margin-top:0.8rem;">
  These results reflect the key behavior of NEM: emissivity differences arise naturally
  by normalizing all radiances with respect to the reference pixel.
  No a priori emissivity information is required.
</p>

<p style="margin-top:1.2rem;">
  To complement this toy example, the following interactive figure shows how emissivity
  influences the <strong>apparent (brightness) temperature</strong> retrieved from radiance.  
  All curves intersect exactly at \(T_s = T_\text{bg}\),
  illustrating the radiative balance point where surface and background emit equally.
</p>


<!-- Contenedor del plot -->
<div id="nem-scatter-interactive" class="alg-figure" style="margin-top:1rem;"></div>

<!-- Slider para T_bg -->
<div style="margin-top:1rem; text-align:center;">
  <label for="TbgRange" style="font-weight:600; font-size:0.95rem;">
    Background temperature \(T_\text{bg}\) (K):
  </label><br>
  <input
    type="range"
    id="TbgRange"
    min="260"
    max="340"
    value="293"
    step="1"
    style="width:350px; accent-color:#ff0000; vertical-align:middle;">
  <div id="TbgValue" style="font-weight:700; font-size:1rem; margin-top:0.4rem;">
    T<sub>bg</sub> = 293 K
  </div>
</div>

<script type="module">
  import * as Plot from "https://cdn.jsdelivr.net/npm/@observablehq/plot@0.6/+esm";

  // --- Physical constants (SI) ---
  const c  = 299792458;
  const h  = 6.62607015e-34;
  const k  = 1.380649e-23;
  const c1 = 2 * h * c * c;   // 2hc^2
  const c2 = (h * c) / k;     // hc/k

  const lambda_um = 10;
  const lambda_m  = lambda_um * 1e-6;

  // Planck radiance B_lambda(T) in W·m⁻²·sr⁻¹·m⁻¹
  function B_lambda(T) {
    const x = c2 / (lambda_m * T);
    return (c1 / Math.pow(lambda_m, 5)) / (Math.exp(x) - 1);
  }

  // Inverse Planck: brightness temperature from radiance
  function Tb_from_L(L) {
    const lnTerm = Math.log(1 + c1 / (Math.pow(lambda_m, 5) * L));
    return c2 / (lambda_m * lnTerm);
  }

  // Ts range for the x-axis
  const Ts_min  = 250;
  const Ts_max  = 350;

  // Emissivities taken from your example
  const epsList = [0.67, 0.80, 0.87, 0.93, 0.99];
  const samples = 200;

  // Build curves Tb vs Ts for a given T_bg
  function buildCurves(T_bg) {
    const curves = [];
    for (const eps of epsList) {
      for (let i = 0; i < samples; i++) {
        const Ts = Ts_min + (Ts_max - Ts_min) * (i / (samples - 1));
        const L  = eps * B_lambda(Ts) + (1 - eps) * B_lambda(T_bg);
        const Tb = Tb_from_L(L);
        curves.push({ Ts, Tb, eps });
      }
    }
    return curves;
  }

  // Label positions for ε on the right side
  function buildLabels(T_bg) {
    const labelTs = Ts_max + 3;
    return epsList.map(eps => {
      const L  = eps * B_lambda(Ts_max) + (1 - eps) * B_lambda(T_bg);
      const Tb = Tb_from_L(L);
      return { Ts: labelTs, Tb, label: eps.toFixed(2) };
    });
  }

  const TbgSlider = document.getElementById("TbgRange");
  const TbgValue  = document.getElementById("TbgValue");

  function renderPlot() {
    const T_bg = parseInt(TbgSlider.value, 10);
    TbgValue.innerHTML = `T<sub>bg</sub> = ${T_bg} K`;

    const curves = buildCurves(T_bg);
    const labels = buildLabels(T_bg);

    const chart = Plot.plot({
      height: 360,
      marginLeft: 70,
      marginRight: 70,
      marginBottom: 45,
      x: {
        label: "Target Temperature T_s (K)",
        domain: [Ts_min, Ts_max + 5]
      },
      y: {
        label: "Apparent Temperature T_b (K)",
        domain: [250, 350],
        grid: true
      },
      marks: [
        // Family of curves for each ε
        Plot.line(curves, {
          x: "Ts",
          y: "Tb",
          stroke: d => d.eps,
          strokeWidth: d => (d.eps === 0.99 ? 3 : 2)
        }),
        // Vertical line at Ts = T_bg (intersection)
        Plot.ruleX([T_bg], { stroke: "#aaa", strokeDasharray: "4 4" }),
        // ε labels on the right side
        Plot.text(labels, {
          x: "Ts",
          y: "Tb",
          text: "label",
          dx: 4,
          dy: 0,
          fontSize: 12,
          fontWeight: 600,
          fill: "#111"
        })
      ]
    });

    const div = document.getElementById("nem-scatter-interactive");
    div.innerHTML = "";
    div.append(chart);
  }

  TbgSlider.addEventListener("input", renderPlot);
  renderPlot(); // initial render
</script>

<h2 style="margin-top:2rem;">References</h2>

<div style="border: 2px solid #ccc; padding: 1rem 1.2rem; background-color: #f9f9f9; border-radius: 8px; width: 80%; margin-bottom: 2rem;">
  <ul style="margin:0; padding-left:1.2rem;">
    <li><strong>Gillespie, A. R.</strong> (1985). Radiometric Calibration of Landsat Thematic Mapper Thermal-Infrared Data. <em>Remote Sensing of Environment</em>, 17(2), 103–117.</li>
    <li><strong>Sobrino, J. A., Jiménez-Muñoz, J. C., & Paolini, L.</strong> (2004). Land Surface Temperature Retrieval from Landsat TM 5. <em>Remote Sensing of Environment</em>, 90(4), 434–440.</li>
    
  </ul>
</div>














  



<!-- MathJax loader (only add once on the page) -->
<script>
window.MathJax = { tex: { inlineMath: [['$', '$'], ['\\(', '\\)']] } };
</script>
<script async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>


</section>
</div>
