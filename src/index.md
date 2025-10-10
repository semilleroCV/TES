---
toc: false
---

<div class="hero">
  <h1>TES</h1>
  <h2>Temperature–Emissivity Separation (TES)</h2>
  <div class="nav-links">
    <a href="/algorithms/about">About</a>
    <a href="/algorithms/documentation">Documentation</a>
    <a href="/algorithms/contact">Contact</a>
  </div>
  </div>

<div class="lead">
  <p>
    Temperature–Emissivity Separation estimates land surface temperature and spectral emissivity
    from thermal infrared imagery by decoupling their intertwined signals. Typical workflows
    include atmospheric correction, emissivity priors (e.g., NDVI-based), and iterative refinement
    to recover both temperature and material-dependent emissivity.
  </p>
</div>



<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 4rem 0 8rem;
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

.nav-links {
  display: flex;
  gap: 1.5rem;
  margin-top: 2rem;
}

.nav-links a {
  padding: 0.75rem 1.5rem;
  background: var(--theme-foreground-focus);
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-weight: 500;
  transition: transform 0.2s, opacity 0.2s;
}

.nav-links a:hover {
  transform: translateY(-2px);
  opacity: 0.9;
}

.card {
  padding: 1.5rem;
  text-align: left;
}

.card h3 {
  margin-top: 0;
  font-size: 1.5rem;
}

.card a {
  font-weight: 500;
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

</style>
