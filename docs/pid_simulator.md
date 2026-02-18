# PID Controller Simulator

A simple interactive simulation of a **first-order process**, an **actuator** with output saturation, and a **PID controller**. Tune the controller parameters and observe how the process responds.

---

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.2/dist/chart.umd.min.js"></script>

<style>
  .pid-app {
    font-family: inherit;
    background: var(--md-default-bg-color, #fff);
    padding: 0;
  }
  .pid-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  @media (max-width: 600px) {
    .pid-grid { grid-template-columns: 1fr; }
  }
  .pid-card {
    background: var(--md-code-bg-color, #f5f5f5);
    border-radius: 8px;
    padding: 1rem;
  }
  .pid-card h3 {
    margin: 0 0 0.75rem 0;
    font-size: 0.95rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: var(--md-primary-fg-color, #1976d2);
    border-bottom: 1px solid var(--md-primary-fg-color, #1976d2);
    padding-bottom: 0.4rem;
  }
  .param-row {
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;
    gap: 0.5rem;
  }
  .param-row label {
    width: 130px;
    font-size: 0.85rem;
    flex-shrink: 0;
  }
  .param-row input[type=range] {
    flex: 1;
    accent-color: var(--md-primary-fg-color, #1976d2);
  }
  .param-row input[type=number] {
    width: 72px;
    padding: 2px 4px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 0.85rem;
    background: var(--md-default-bg-color, #fff);
    color: var(--md-default-fg-color, #000);
  }
  .pid-gauges {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
    flex-wrap: wrap;
  }
  .gauge-box {
    flex: 1;
    min-width: 100px;
    background: var(--md-code-bg-color, #f5f5f5);
    border-radius: 8px;
    padding: 0.75rem 1rem;
    text-align: center;
  }
  .gauge-box .gauge-label {
    font-size: 0.75rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    opacity: 0.7;
  }
  .gauge-box .gauge-value {
    font-size: 1.6rem;
    font-weight: 700;
    line-height: 1.2;
  }
  .gauge-pv  .gauge-value { color: #2196f3; }
  .gauge-sp  .gauge-value { color: #ff9800; }
  .gauge-co  .gauge-value { color: #4caf50; }
  .gauge-err .gauge-value { color: #e53935; }
  .pid-btn-row {
    display: flex;
    gap: 0.75rem;
    margin-bottom: 1rem;
    align-items: center;
    flex-wrap: wrap;
  }
  .pid-btn {
    padding: 0.45rem 1.2rem;
    border: none;
    border-radius: 6px;
    font-size: 0.9rem;
    cursor: pointer;
    font-weight: 600;
    transition: opacity 0.15s;
  }
  .pid-btn:hover { opacity: 0.85; }
  .btn-start  { background: #4caf50; color: #fff; }
  .btn-stop   { background: #e53935; color: #fff; }
  .btn-reset  { background: #757575; color: #fff; }
  .btn-tune   { background: #7b1fa2; color: #fff; }
  .tune-select {
    padding: 0.4rem 0.6rem;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 0.85rem;
    background: var(--md-default-bg-color, #fff);
    color: var(--md-default-fg-color, #000);
    cursor: pointer;
  }
  .tune-result {
    display: none;
    margin-top: 0.6rem;
    padding: 0.5rem 0.75rem;
    background: rgba(123,31,162,0.08);
    border-left: 3px solid #7b1fa2;
    border-radius: 4px;
    font-size: 0.82rem;
    line-height: 1.6;
  }
  .pid-chart-wrap {
    position: relative;
    height: 280px;
  }
</style>

<div class="pid-app">

  <!-- Live value gauges -->
  <div class="pid-gauges">
    <div class="gauge-box gauge-pv">
      <div class="gauge-label">Process Value (PV)</div>
      <div class="gauge-value" id="g-pv">0.0</div>
    </div>
    <div class="gauge-box gauge-sp">
      <div class="gauge-label">Setpoint (SP)</div>
      <div class="gauge-value" id="g-sp">50.0</div>
    </div>
    <div class="gauge-box gauge-co">
      <div class="gauge-label">Control Output (CO)</div>
      <div class="gauge-value" id="g-co">0.0</div>
    </div>
    <div class="gauge-box gauge-err">
      <div class="gauge-label">Error (SP–PV)</div>
      <div class="gauge-value" id="g-err">50.0</div>
    </div>
  </div>

  <!-- Trend chart -->
  <div class="pid-chart-wrap">
    <canvas id="pidChart"></canvas>
  </div>

  <br>

  <!-- Controls -->
  <div class="pid-btn-row">
    <button class="pid-btn btn-start" id="btnStart" onclick="pidStart()">&#9654; Start</button>
    <button class="pid-btn btn-stop"  id="btnStop"  onclick="pidStop()"  disabled>&#9646;&#9646; Stop</button>
    <button class="pid-btn btn-reset"               onclick="pidReset()">&#8635; Reset</button>
    <span id="sim-time-label" style="font-size:0.85rem;opacity:0.7;">t = 0 s</span>
  </div>

  <div class="pid-grid">

    <!-- Setpoint -->
    <div class="pid-card">
      <h3>Setpoint</h3>
      <div class="param-row">
        <label>SP (0–100)</label>
        <input type="range" id="sl-sp" min="0" max="100" step="1" value="50"
               oninput="document.getElementById('n-sp').value=this.value">
        <input type="number" id="n-sp" min="0" max="100" step="1" value="50"
               oninput="document.getElementById('sl-sp').value=this.value">
      </div>
    </div>

    <!-- PID Parameters -->
    <div class="pid-card">
      <h3>PID Parameters</h3>
      <div class="param-row">
        <label>Kp (gain)</label>
        <input type="range" id="sl-kp" min="0" max="5" step="0.05" value="1.2"
               oninput="document.getElementById('n-kp').value=this.value">
        <input type="number" id="n-kp" min="0" max="20" step="0.05" value="1.2"
               oninput="document.getElementById('sl-kp').value=Math.min(5,this.value)">
      </div>
      <div class="param-row">
        <label>Ki (integral)</label>
        <input type="range" id="sl-ki" min="0" max="2" step="0.01" value="0.3"
               oninput="document.getElementById('n-ki').value=this.value">
        <input type="number" id="n-ki" min="0" max="10" step="0.01" value="0.3"
               oninput="document.getElementById('sl-ki').value=Math.min(2,this.value)">
      </div>
      <div class="param-row">
        <label>Kd (derivative)</label>
        <input type="range" id="sl-kd" min="0" max="2" step="0.01" value="0.05"
               oninput="document.getElementById('n-kd').value=this.value">
        <input type="number" id="n-kd" min="0" max="10" step="0.01" value="0.05"
               oninput="document.getElementById('sl-kd').value=Math.min(2,this.value)">
      </div>
      <div class="param-row" style="margin-top:0.75rem;">
        <label>Auto-tune method</label>
        <select class="tune-select" id="tune-method">
          <option value="simc">SIMC (recommended)</option>
          <option value="zn">Ziegler-Nichols OL</option>
        </select>
        <button class="pid-btn btn-tune" onclick="pidAutoTune()">&#9881; Tune</button>
      </div>
      <div class="tune-result" id="tune-result"></div>
    </div>

    <!-- Process Model -->
    <div class="pid-card">
      <h3>Process Model</h3>
      <div class="param-row">
        <label>Gain (Kprocess)</label>
        <input type="range" id="sl-pg" min="0.1" max="3" step="0.05" value="1.0"
               oninput="document.getElementById('n-pg').value=this.value">
        <input type="number" id="n-pg" min="0.1" max="10" step="0.05" value="1.0"
               oninput="document.getElementById('sl-pg').value=Math.min(3,this.value)">
      </div>
      <div class="param-row">
        <label>Time constant τ (s)</label>
        <input type="range" id="sl-tau" min="1" max="60" step="1" value="10"
               oninput="document.getElementById('n-tau').value=this.value">
        <input type="number" id="n-tau" min="1" max="120" step="1" value="10"
               oninput="document.getElementById('sl-tau').value=Math.min(60,this.value)">
      </div>
      <div class="param-row">
        <label>Dead time θ (s)</label>
        <input type="range" id="sl-dt" min="0" max="20" step="1" value="2"
               oninput="document.getElementById('n-dt').value=this.value">
        <input type="number" id="n-dt" min="0" max="60" step="1" value="2"
               oninput="document.getElementById('sl-dt').value=Math.min(20,this.value)">
      </div>
    </div>

    <!-- Actuator -->
    <div class="pid-card">
      <h3>Actuator</h3>
      <div class="param-row">
        <label>Output min (%)</label>
        <input type="range" id="sl-amin" min="-100" max="0" step="1" value="0"
               oninput="document.getElementById('n-amin').value=this.value">
        <input type="number" id="n-amin" min="-100" max="100" step="1" value="0"
               oninput="document.getElementById('sl-amin').value=Math.max(-100,this.value)">
      </div>
      <div class="param-row">
        <label>Output max (%)</label>
        <input type="range" id="sl-amax" min="0" max="200" step="1" value="100"
               oninput="document.getElementById('n-amax').value=this.value">
        <input type="number" id="n-amax" min="0" max="200" step="1" value="100"
               oninput="document.getElementById('sl-amax').value=Math.min(200,this.value)">
      </div>
      <div class="param-row">
        <label>Rate limit (%/s)</label>
        <input type="range" id="sl-rate" min="1" max="100" step="1" value="50"
               oninput="document.getElementById('n-rate').value=this.value">
        <input type="number" id="n-rate" min="1" max="500" step="1" value="50"
               oninput="document.getElementById('sl-rate').value=Math.min(100,this.value)">
      </div>
    </div>

  </div>
</div>

<script>
(function () {
  // ── Simulation state ─────────────────────────────────────────────────────
  const DT     = 0.1;   // simulation time step (s)
  const WINDOW = 120;   // seconds of history shown in chart
  const MAX_PTS = Math.round(WINDOW / DT);

  let simTimer  = null;
  let simTime   = 0;
  let pv        = 0;
  let integral  = 0;
  let prevError = 0;
  let prevCO    = 0;

  // Dead-time buffer
  let deadBuf   = [];
  let deadSteps = 0;

  // Chart data arrays  (ring-buffer approach — keep last MAX_PTS)
  const labels = [];
  const dataPV = [];
  const dataSP = [];
  const dataCO = [];

  // ── Helpers ──────────────────────────────────────────────────────────────
  function v(id)   { return parseFloat(document.getElementById(id).value); }
  function fmt(x)  { return x.toFixed(1); }
  function clamp(x, lo, hi) { return Math.max(lo, Math.min(hi, x)); }

  // ── Chart setup ──────────────────────────────────────────────────────────
  const ctx  = document.getElementById('pidChart').getContext('2d');
  const chart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: labels,
      datasets: [
        {
          label: 'PV – Process Value',
          data: dataPV,
          borderColor: '#2196f3',
          backgroundColor: 'rgba(33,150,243,0.08)',
          borderWidth: 2,
          pointRadius: 0,
          tension: 0.3,
          fill: false,
          yAxisID: 'y',
        },
        {
          label: 'SP – Setpoint',
          data: dataSP,
          borderColor: '#ff9800',
          backgroundColor: 'transparent',
          borderWidth: 2,
          borderDash: [6, 3],
          pointRadius: 0,
          tension: 0,
          fill: false,
          yAxisID: 'y',
        },
        {
          label: 'CO – Control Output',
          data: dataCO,
          borderColor: '#4caf50',
          backgroundColor: 'rgba(76,175,80,0.06)',
          borderWidth: 2,
          pointRadius: 0,
          tension: 0.3,
          fill: false,
          yAxisID: 'y2',
        },
      ],
    },
    options: {
      animation: false,
      responsive: true,
      maintainAspectRatio: false,
      interaction: { mode: 'index', intersect: false },
      plugins: {
        legend: { position: 'top', labels: { usePointStyle: true, boxWidth: 10, padding: 14 } },
        tooltip: {
          callbacks: {
            label: ctx => `${ctx.dataset.label}: ${ctx.parsed.y.toFixed(2)}`
          }
        }
      },
      scales: {
        x: {
          title: { display: true, text: 'Time (s)' },
          ticks: { maxTicksLimit: 10, maxRotation: 0 },
        },
        y: {
          title: { display: true, text: 'PV / SP' },
          min: -10, max: 110,
          position: 'left',
        },
        y2: {
          title: { display: true, text: 'CO (%)' },
          min: -10, max: 110,
          position: 'right',
          grid: { drawOnChartArea: false },
        },
      },
    },
  });

  // ── Simulation step ───────────────────────────────────────────────────────
  function step() {
    simTime += DT;

    const sp       = v('n-sp');
    const Kp       = v('n-kp');
    const Ki       = v('n-ki');
    const Kd       = v('n-kd');
    const Kproc    = v('n-pg');
    const tau      = Math.max(0.1, v('n-tau'));
    const deadTime = Math.max(0, v('n-dt'));
    const aMin     = v('n-amin');
    const aMax     = v('n-amax');
    const rateMax  = v('n-rate');

    // ── PID ──────────────────────────────────────────────────────────────
    const error  = sp - pv;
    integral    += error * DT;
    const deriv  = (error - prevError) / DT;
    prevError    = error;

    let rawCO = Kp * error + Ki * integral + Kd * deriv;

    // Anti-windup: if CO saturated, stop integrating in that direction
    const satCO = clamp(rawCO, aMin, aMax);
    if (rawCO !== satCO) {
      integral -= error * DT;   // undo last integration
    }

    // ── Actuator: rate limiting then saturation ───────────────────────────
    const maxDelta = rateMax * DT;
    let co = prevCO + clamp(satCO - prevCO, -maxDelta, maxDelta);
    co = clamp(co, aMin, aMax);
    prevCO = co;

    // ── Dead-time buffer ──────────────────────────────────────────────────
    const newDeadSteps = Math.round(deadTime / DT);
    if (newDeadSteps !== deadSteps) {
      deadSteps = newDeadSteps;
      if (deadBuf.length > deadSteps + 1) deadBuf = deadBuf.slice(-(deadSteps + 1));
    }
    deadBuf.push(co);
    const delayedCO = deadBuf.length > deadSteps ? deadBuf[deadBuf.length - 1 - deadSteps] : 0;

    // ── First-order process: τ·ẋ = Kp·u − x  →  Euler ───────────────────
    pv += (DT / tau) * (Kproc * delayedCO - pv);
    pv  = Math.max(-200, Math.min(200, pv));

    // ── Update chart data ─────────────────────────────────────────────────
    const tLabel = simTime.toFixed(1);
    labels.push(tLabel);
    dataPV.push(parseFloat(pv.toFixed(2)));
    dataSP.push(sp);
    dataCO.push(parseFloat(co.toFixed(2)));

    if (labels.length > MAX_PTS) {
      labels.shift();
      dataPV.shift();
      dataSP.shift();
      dataCO.shift();
    }

    chart.update('none');   // no animation for real-time feel

    // ── Gauges ────────────────────────────────────────────────────────────
    document.getElementById('g-pv').textContent  = fmt(pv);
    document.getElementById('g-sp').textContent  = fmt(sp);
    document.getElementById('g-co').textContent  = fmt(co);
    document.getElementById('g-err').textContent = fmt(sp - pv);
    document.getElementById('sim-time-label').textContent = `t = ${simTime.toFixed(1)} s`;
  }

  // ── Controls ─────────────────────────────────────────────────────────────
  window.pidStart = function () {
    if (simTimer) return;
    simTimer = setInterval(step, DT * 1000);
    document.getElementById('btnStart').disabled = true;
    document.getElementById('btnStop').disabled  = false;
  };

  window.pidStop = function () {
    clearInterval(simTimer);
    simTimer = null;
    document.getElementById('btnStart').disabled = false;
    document.getElementById('btnStop').disabled  = true;
  };

  // ── Auto-tuner ────────────────────────────────────────────────────────────
  window.pidAutoTune = function () {
    const K   = Math.max(0.001, v('n-pg'));  // process gain
    const tau = Math.max(0.1,   v('n-tau')); // time constant
    const th  = Math.max(0,     v('n-dt'));  // dead time
    const method = document.getElementById('tune-method').value;

    let Kp, Ki, Kd, label;

    if (method === 'simc') {
      // SIMC (Skogestad IMC) — tight setting: τc = max(0.1τ, 0.8θ)
      const tc = Math.max(0.1 * tau, 0.8 * th || 0.1 * tau);
      Kp = (1 / K) * (tau / (tc + th));
      const Ti = Math.min(tau, 4 * (tc + th));
      const Td = th / 2;
      Ki = Kp / Ti;
      Kd = Kp * Td;
      label = `SIMC  (τc = ${tc.toFixed(2)} s)`;
    } else {
      // Ziegler-Nichols open-loop (requires dead time > 0)
      if (th < 0.01) {
        document.getElementById('tune-result').style.display = 'block';
        document.getElementById('tune-result').innerHTML =
          '&#9888; Ziegler-Nichols requires dead time &theta; &gt; 0. Set &theta; &ge; 1 s and try again.';
        return;
      }
      Kp = (1.2 / K) * (tau / th);
      const Ti = 2 * th;
      const Td = 0.5 * th;
      Ki = Kp / Ti;
      Kd = Kp * Td;
      label = 'Ziegler-Nichols OL';
    }

    // Round to 3 decimal places
    Kp = Math.round(Kp * 1000) / 1000;
    Ki = Math.round(Ki * 1000) / 1000;
    Kd = Math.round(Kd * 1000) / 1000;

    // Apply to controls
    function setParam(sliderId, numberId, val, lo, hi) {
      const clamped = Math.min(hi, Math.max(lo, val));
      document.getElementById(numberId).value = val;   // allow wider range in number
      document.getElementById(sliderId).value = clamped;
    }
    setParam('sl-kp', 'n-kp', Kp, 0, 5);
    setParam('sl-ki', 'n-ki', Ki, 0, 2);
    setParam('sl-kd', 'n-kd', Kd, 0, 2);

    // Reset integrator state so new parameters take effect cleanly
    integral = 0; prevError = 0;

    const res = document.getElementById('tune-result');
    res.style.display = 'block';
    res.innerHTML = `<strong>${label}</strong><br>
      Kp = <strong>${Kp}</strong> &nbsp;|
      Ki = <strong>${Ki}</strong> &nbsp;|
      Kd = <strong>${Kd}</strong>`;
  };

  window.pidReset = function () {
    pidStop();
    simTime = 0; pv = 0; integral = 0; prevError = 0; prevCO = 0;
    deadBuf = []; deadSteps = 0;
    labels.length = 0; dataPV.length = 0; dataSP.length = 0; dataCO.length = 0;
    chart.update();
    document.getElementById('g-pv').textContent  = '0.0';
    document.getElementById('g-sp').textContent  = fmt(v('n-sp'));
    document.getElementById('g-co').textContent  = '0.0';
    document.getElementById('g-err').textContent = fmt(v('n-sp'));
    document.getElementById('sim-time-label').textContent = 't = 0 s';
  };
})();
</script>

---

## How it works

| Block | Model |
|---|---|
| **PID Controller** | Standard parallel form: $u = K_p e + K_i \int e \, dt + K_d \dot{e}$, with **anti-windup** (conditional integration) |
| **Actuator** | Output **rate limiter** (max %/s) followed by a hard **saturation clamp** (min/max %) |
| **Process** | First-order lag: $\tau \dot{x} = K_{proc} \cdot u_{delayed} - x$, with configurable **dead time θ** |
| **SIMC auto-tune** | $\tau_c = \max(0.1\tau,\, 0.8\theta)$, then $K_p = \frac{1}{K}\frac{\tau}{\tau_c+\theta}$, $T_i = \min(\tau,\, 4(\tau_c+\theta))$, $T_d = \theta/2$ |
| **ZN OL auto-tune** | $K_p = \frac{1.2}{K}\frac{\tau}{\theta}$, $T_i = 2\theta$, $T_d = 0.5\theta$ (requires $\theta > 0$) |

### Tips for tuning

- Start with **Ki = 0** and **Kd = 0** and increase **Kp** until the response is fast but not too oscillatory.
- Add a small **Ki** to eliminate steady-state offset.
- Add a small **Kd** to reduce overshoot — watch out for noise amplification.
- Increase **dead time θ** and/or **τ** to make the process harder to control.
