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
    <p>Minimum Mean Difference approach for TES.</p>
  </header>

  <section class="alg-meta">
    <div><strong>Authors</strong>: Jane Doe, Chris Example</div>
    <span class="sep"></span>
    <div><strong>Date</strong>: 1994</div>
  </section>

  

  <section class="alg-section alg-callout">
    <h2>Key Idea</h2>
    <p>Alternate between temperature update and emissivity smoothing.</p>
  </section>

  <section class="alg-section">
    <h2>Steps</h2>
    <ol>
      <li>Initialize ${tex`T`} and ${tex`\varepsilon`}.</li>
      <li>Compute forward model and residuals.</li>
      <li>Update ${tex`T`} to reduce residual mean; smooth ${tex`\varepsilon`}.</li>
    </ol>
  </section>

  <section class="alg-section alg-refs">
    <h3>References</h3>
    <ol>
      <li>Author, C., MMD for TES, Journal, 1994.</li>
    </ol>
  </section>
</div>
