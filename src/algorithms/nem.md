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
    <p style="margin-bottom:1rem;">
  The Normalized Emissivity Method (NEM) is used to estimate land surface temperature and spectral emissivity from thermal imagery, particularly in cases where prior knowledge of material emissivity is unavailable. This method enables reliable retrievals of surface temperature and emissivity even when only a limited number of thermal bands are available.
</p>

<p style="margin-bottom:1rem;">
  The approach is based on the assumption that within the set of thermal bands, at least one band exhibits the maximum emissivity (<strong>ε<sub>max</sub></strong>).
  From this reference band, the <strong>brightness temperature</strong> is estimated using the inverse of Planck’s law, which serves as the basis for deriving the kinetic temperature and emissivity of the remaining pixels.
</p>

  </br>

  


  <h3 style="margin:1rem 0 0.5rem">Brightness Temperature</h3>
  <p>
  The brightness temperature (<strong>T<sub>b</sub></strong>) can be derived from the
  measured spectral radiance (<strong>L<sub>λ</sub></strong>) using the inverse of Planck’s law:
  </p>
  
  <p style="text-align:center; color:#555; margin-top:0;">
  \( T_b = \dfrac{c_2}{\lambda \,\ln\!\left(1 + \dfrac{c_1}{\lambda^5 L_\lambda}\right)} \), 
  where \(c_1 = 2hc^2\) and \(c_2 = \dfrac{hc}{k}\).
  </p>


   
  

  <!-- Contenedor donde irá el gráfico -->
  <div id="nem-plot" class="alg-figure bleed" style="margin-top:1rem;">
  </div>

  <!-- Controles interactivos -->
  <div style="margin-top:2rem; text-align:center;">
    <label for="tempRange" style="font-weight:600; font-size:0.95rem;">Surface Temperature (K):</label><br>
    <input 
      type="range" 
      id="tempRange" 
      min="300" 
      max="2000" 
      value="300" 
      step="1"
      style="width:350px; accent-color:#ff0000; vertical-align:middle;">
    <div id="tempValue" style="font-weight:700; font-size:1rem; margin-top:0.4rem;">300 K</div>
  </div>

  <script type="module">
    import * as Plot from "https://cdn.jsdelivr.net/npm/@observablehq/plot@0.6/+esm";

    // --- Constantes físicas ---
    const c = 299792458;       // m/s
    const h = 6.62607015e-34;  // J·s
    const k = 1.380649e-23;    // J/K

    // --- Parámetros base ---
    const lambda_um = 10;      // µm
    const lambda_m = lambda_um * 1e-6;
    const samples = 200;

    // --- Función Tb(eps) ---
    function brightnessTemp(eps, T, lambda_m) {
      const exponent = (h * c) / (lambda_m * k * T);
      return (h * c) / (lambda_m * k * Math.log(1 + (Math.exp(exponent) - 1) / eps));
    }

    // --- Generar datos ---
    function generateData(T) {
      return Array.from({ length: samples }, (_, i) => {
        const eps = 0.3 + 0.7 * (i / (samples - 1));
        return { epsilon: eps, Tb: brightnessTemp(eps, T, lambda_m) };
      });
    }

    // --- Dibujar gráfico ---
    function renderPlot(T) {
      const data = generateData(T);

      const chart = Plot.plot({
        height: 350,
        marginLeft: 70,
        marginBottom: 45,
        x: { label: "Emissivity (ε)", domain: [0.3, 1.0] },
        y: { label: "Brightness Temperature (K)", grid: true },
        marks: [
          Plot.line(data, { x: "epsilon", y: "Tb", stroke: "#f30606ff", strokeWidth: 2 }),
          Plot.ruleY([T], { stroke: "#666", strokeDasharray: "4 4" }),
          Plot.text(
            [{ epsilon: 0.95, Tb: T, label: `Tₛ = ${T} K` }],
            { x: "epsilon", y: "Tb", text: "label", fontSize: 12, dy: -6, fill: "#333" }
          )
        ]
      });

      const div = document.getElementById("nem-plot");
      div.innerHTML = "";
      div.append(chart);
    }

    // --- Interactividad ---
    const tempSlider = document.getElementById("tempRange");
    const tempValue = document.getElementById("tempValue");

    function updatePlot() {
      const T = parseInt(tempSlider.value);
      tempValue.textContent = `${T} K`;
      renderPlot(T);
    }

    tempSlider.addEventListener("input", updatePlot);
    updatePlot(); // inicializa
  </script>
  
  <p style="margin-top:1rem;">
  Once the kinetic temperature (<strong>T<sub>k</sub></strong>) has been identified as the highest
  brightness temperature, the next step is to estimate the spectral emissivity
  (<strong>ε<sub>i</sub></strong>) for each band or pixel.
  Using the measured spectral radiance (<strong>L<sub>i</sub></strong>), the emissivity can be derived
  from Planck’s radiance function at the kinetic temperature as:
</p>

<p style="text-align:center; font-size:1.05rem; margin-top:0.6rem;">
  \[
    \varepsilon_i = \frac{L_i}{B(\lambda, T_k)}
  \]
</p>

<p style="margin-top:0.6rem;">
  Here, \(B(\lambda, T_k)\) represents the blackbody spectral radiance computed from Planck’s law
  for the wavelength \( \lambda \) and kinetic temperature \( T_k \).
  This normalization step ensures that emissivity values remain consistent across all thermal bands.
</p>
<p style="margin-top:1rem;">
  With both the kinetic temperature (<strong>T<sub>k</sub></strong>) and the emissivity values
  (<strong>ε<sub>i</sub></strong>) estimated, the NEM method allows the retrieval of a
  consistent temperature–emissivity set for each pixel in the scene.
  These results can then be refined iteratively or compared with other
  temperature–emissivity separation (TES) techniques to improve accuracy.
</p>


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
  Consider a small set of 5 pixels with measured radiances (<strong>L</strong>) at a wavelength of 10 μm:
</p>

<table style="width:50%; border-collapse: collapse; margin: 0.5rem 0; font-size: 0.95rem;">
  <thead>
    <tr style="background-color:#f3f4f6;">
      <th style="border: 1px solid #ccc; padding: 6px 10px; text-align:center;">Pixel</th>
      <th style="border: 1px solid #ccc; padding: 6px 10px; text-align:center;">L (W/(m²·sr·μm))</th>
    </tr>
  </thead>
  <tbody>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">1</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">10</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">2</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">12</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">3</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">15</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">4</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">14</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">5</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">13</td></tr>
  </tbody>
</table>

<ol style="margin-top:0.6rem; padding-left:1.2rem; font-size:0.95rem;">
  <li>The pixel with maximum radiance (Pixel 3) defines the kinetic temperature <strong>T<sub>k</sub></strong>. Using Planck’s law, suppose T<sub>k</sub> = 330 K.</li>
  <li>Emissivities of the other pixels are calculated relative to this maximum:</li>
</ol>

<table style="width:50%; border-collapse: collapse; margin: 0.5rem 0 1rem 0; font-size: 0.95rem;">
  <thead>
    <tr style="background-color:#f3f4f6;">
      <th style="border: 1px solid #ccc; padding: 6px 10px; text-align:center;">Pixel</th>
      <th style="border: 1px solid #ccc; padding: 6px 10px; text-align:center;">ε</th>
    </tr>
  </thead>
  <tbody>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">1</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">0.67</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">2</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">0.80</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">3</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">1.00</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">4</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">0.93</td></tr>
    <tr><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">5</td><td style="border: 1px solid #ccc; padding: 4px 8px; text-align:center;">0.87</td></tr>
  </tbody>
</table>

<p style="font-size:0.95rem;">
  This small example illustrates how NEM can retrieve both kinetic temperature and emissivity values using only thermal radiances, without prior knowledge of material properties.
</p>

<!-- Contenedor del gráfico -->
<div id="nem-scatter-interactive" class="alg-figure" style="margin-top:1rem;"></div>

<script type="module">
  import * as Plot from "https://cdn.jsdelivr.net/npm/@observablehq/plot@0.6/+esm";

  // Datos base
  const data = [
    { pixel: 1, L: 10, eps: 0.67 },
    { pixel: 2, L: 12, eps: 0.80 },
    { pixel: 3, L: 15, eps: 1.00 },
    { pixel: 4, L: 14, eps: 0.93 },
    { pixel: 5, L: 13, eps: 0.87 },
  ];

  // Crear gráfico
  const chart = Plot.plot({
    height: 340,
    marginLeft: 65,
    marginBottom: 45,
    x: { label: "Emissivity (ε)", domain: [0.6, 1.05] },
    y: { label: "Radiance (W/(m²·sr·μm))", domain: [9, 16], grid: true },
    marks: [
      Plot.line(data, { x: "eps", y: "L", stroke: "#eb0808ff", strokeWidth: 2 }),
      Plot.dot(data, { 
        x: "eps", 
        y: "L", 
        r: 5, 
        fill: "#f50b0bff", 
        title: d => 
          `Pixel: ${d.pixel}\nε: ${d.eps.toFixed(2)}\nL: ${d.L.toFixed(2)} W/(m²·sr·μm)`
      }),
      Plot.text(data, { 
        x: "eps", 
        y: "L", 
        text: d => `${d.pixel}`, 
        dy: -10, 
        fill: "#111", 
        fontSize: 12, 
        fontWeight: 600 
      })
    ],
  });

  document.getElementById("nem-scatter-interactive").append(chart);
</script>

<img src="assets/nempaper.png" alt="Paper original" width="500">

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

