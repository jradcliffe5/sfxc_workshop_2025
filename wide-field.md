### A1. Calculate wide-field correlation parameters

<!-- Interactive FoV Calculator -->
<div class="card" id="fov-calc" style="padding:1rem; margin-top:1rem;">
  <h3 class="tight">Interactive FoV Calculator</h3>
  <p class="soft">Compute upper-limit FoVs (arcsec) for bandwidth smearing (two forms) and time smearing. Uses site styles; only minimal local styles below.</p>

  <style>
    .fov-form { display:grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
    .fov-field { display:flex; flex-direction:column; gap:.35rem; }
    .fov-field label { color: var(--muted); font-size:.95rem; }
    .fov-pill { display:inline-block; padding:.1rem .5rem; border:1px solid var(--border); border-radius:999px; font-size:.8rem; color:var(--muted); }
    #fov-calc input[type="number"]{
      width:100%; padding:.66rem .8rem; border-radius: calc(var(--radius, 12px) - 6px);
      border:1px solid var(--border); background: var(--bg-soft); color: var(--text);
      outline:none; appearance:textfield;
    }
    #fov-calc input[type="number"]:focus{
      border-color: color-mix(in oklab, var(--accent, #7da3ff) 50%, var(--border));
      box-shadow: 0 0 0 3px color-mix(in oklab, var(--accent, #7da3ff) 15%, transparent);
    }
    .fov-hint { color: var(--muted); font-size:.9rem; }
    .fov-sep { height:1px; background: var(--border); margin:.6rem 0; grid-column:1/-1; }
    .fov-row { display:flex; align-items:baseline; gap:.35rem; flex-wrap:wrap; }
    .fov-value { font-variant-numeric: tabular-nums; font-weight:800; font-size:1.6rem; }
    .fov-unit { color: var(--muted); }
    .fov-btn {
      appearance:none; border:1px solid var(--border); background: color-mix(in oklab, var(--bg-soft) 85%, transparent);
      color: var(--text); padding:.55rem .9rem; border-radius: 10px; cursor:pointer;
    }
    .fov-btn:hover { border-color: color-mix(in oklab, var(--accent, #7da3ff) 50%, var(--border)); }
    .fov-small { font-size:.9rem; }
  </style>

  <form id="fov-form" class="fov-form" autocomplete="off">
    <div class="fov-field">
      <label for="B_km">Baseline length B <span class="fov-pill">km</span></label>
      <input type="number" id="B_km" step="1" min="0" value="2500" />
      <div class="fov-hint">Projected baseline. For bandwidth formulas, B is in units of 1000&nbsp;km.</div>
    </div>
    <div class="fov-field">
      <label for="lambda_m">Wavelength λ <span class="fov-pill">m</span></label>
      <input type="number" id="lambda_m" step="0.0001" min="0" value="0.21" />
      <div class="fov-hint">Example: 0.21 m (≈ 1.4 GHz)</div>
    </div>

    <div class="fov-sep"></div>

    <div class="fov-field">
      <label for="Nnu">Number of channels N<sub>ν</sub></label>
      <input type="number" id="Nnu" step="1" min="1" value="2048" />
    </div>
    <div class="fov-field">
      <label for="BW_SB">Sub-bandwidth BW<sub>SB</sub> <span class="fov-pill">MHz</span></label>
      <input type="number" id="BW_SB" step="0.01" min="0.001" value="32" />
    </div>
    <div class="fov-field">
      <label for="NSB">Number of sub-bands N<sub>SB</sub></label>
      <input type="number" id="NSB" step="1" min="1" value="1" />
    </div>
    <div class="fov-field">
      <label for="BW_tot">Total bandwidth BW<sub>tot</sub> <span class="fov-pill">MHz</span></label>
      <input type="number" id="BW_tot" step="0.01" min="0.001" value="32" />
      <div class="fov-hint">If BW<sub>tot</sub> = N<sub>SB</sub> · BW<sub>SB</sub>, both bandwidth forms match.</div>
    </div>

    <div class="fov-sep"></div>

    <div class="fov-field">
      <label for="tint">Integration time t<sub>int</sub> <span class="fov-pill">s</span></label>
      <input type="number" id="tint" step="0.01" min="0.001" value="2.0" />
    </div>

    <div class="fov-field" style="align-self:end">
      <button type="button" id="resetFov" class="fov-btn">Reset to example</button>
      <span class="fov-hint">B=2500 km, Nν=2048, BW<sub>SB</sub>=32 MHz</span>
    </div>
  </form>

  <div class="fov-sep"></div>

  <div class="steps" style="--step-prefix:'· ';">
    <div class="step">
      <h4 class="tight">Bandwidth smearing (Method 1)</h4>
      <div class="fov-row"><span class="fov-value" id="fovBW1">—</span><span class="fov-unit">arcsec</span></div>
      <p class="soft fov-small">49.5″ · (1000/B<sub>km</sub>) · (N<sub>ν</sub>/BW<sub>SB</sub>)</p>
    </div>

    <div class="step">
      <h4 class="tight">Bandwidth smearing (Method 2)</h4>
      <div class="fov-row"><span class="fov-value" id="fovBW2">—</span><span class="fov-unit">arcsec</span></div>
      <p class="soft fov-small">49.5″ · (1000/B<sub>km</sub>) · (N<sub>SB</sub>N<sub>ν</sub>/BW<sub>tot</sub>)</p>
    </div>

    <div class="step">
      <h4 class="tight">Time smearing</h4>
      <div class="fov-row"><span class="fov-value" id="fovTime">—</span><span class="fov-unit">arcsec</span></div>
      <p class="soft fov-small">18.56″ · (λ/B) · (1/t<sub>int</sub>) → implemented as 18.56″ · (λ / (B<sub>km</sub>·1000)) · (1/t)</p>
    </div>

    <div class="step">
      <h4 class="tight">Quick check vs. table</h4>
      <p class="soft fov-small">With B=2500 km, Nν=2048, BW<sub>SB</sub>=32 MHz ⇒ Method 1 should be 1267.20″. Computed: <span class="fov-pill" id="tableCheck">—</span></p>
    </div>
  </div>
</div>

<script>
(function(){
  const $ = id => document.getElementById(id);
  const fields = ["B_km","lambda_m","Nnu","BW_SB","NSB","BW_tot","tint"].map($);

  function fmt(x){
    if (!isFinite(x)) return "—";
    if (x === 0) return "0.00";
    const abs = Math.abs(x);
    return (abs >= 0.01 && abs < 1e5) ? x.toFixed(2) : x.toExponential(2);
  }

  function calc(){
    const B_km   = parseFloat($("B_km").value);
    const lambda = parseFloat($("lambda_m").value);
    const Nnu    = parseFloat($("Nnu").value);
    const BW_SB  = parseFloat($("BW_SB").value);
    const NSB    = parseFloat($("NSB").value);
    const BWtot  = parseFloat($("BW_tot").value);
    const tint   = parseFloat($("tint").value);

    const valid = [B_km,lambda,Nnu,BW_SB,NSB,BWtot,tint].every(v => isFinite(v) && v > 0);
    if(!valid){
      ["fovBW1","fovBW2","fovTime","tableCheck"].forEach(id => $(id).textContent = "—");
      return;
    }

    const factorB = 49.5 * (1000.0 / B_km);
    const fovBW1 = factorB * (Nnu / BW_SB);
    const fovBW2 = factorB * ((NSB * Nnu) / BWtot);

    const fovTime = 18.56 * (lambda / (B_km * 1000.0)) * (1.0 / tint);

    $("fovBW1").textContent = fmt(fovBW1);
    $("fovBW2").textContent = fmt(fovBW2);
    $("fovTime").textContent = fmt(fovTime);

    const check = 49.5 * (1000.0 / 2500.0) * (2048 / 32.0);
    $("tableCheck").textContent = fmt(check);
  }

  $("resetFov").addEventListener("click", () => {
    $("B_km").value = 2500;
    $("lambda_m").value = 0.21;
    $("Nnu").value = 2048;
    $("BW_SB").value = 32;
    $("NSB").value = 1;
    $("BW_tot").value = 32;
    $("tint").value = 2.0;
    calc();
  });

  fields.forEach(el => el.addEventListener("input", calc));
  calc();
})();
</script>
