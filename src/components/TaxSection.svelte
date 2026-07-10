<script>
  import csvRaw from '../data/lead-tax-scenarios.csv?raw';

  const lines = csvRaw.trim().split('\n').slice(1);
  const rows = lines.map(l => {
    const [scenario, year, fee, demand, revenue, ...behaviorParts] = l.split(',');
    return {
      scenario,
      year: parseInt(year),
      fee: parseFloat(fee),
      demand: parseFloat(demand),
      revenue: parseFloat(revenue),
      behavior: behaviorParts.join(',').replace(/^"|"$/g, ''),
    };
  });

  const scenarios = { soft: [], notax: [] };
  for (const r of rows) scenarios[r.scenario].push(r);

  const years = scenarios.soft.map(r => r.year);
  const colors = { soft: '#D94A4A', notax: '#999999' };

  const W = 600, H = 240, padL = 44, padR = 12, padT = 12, padB = 34;
  const gW = W - padL - padR, gH = H - padT - padB;
  const x = (i) => Math.round(padL + (i / (years.length - 1)) * gW);
  const y = (v) => Math.round(padT + gH - (v / 100) * gH);

  const gridY = [0, 25, 50, 75, 100];
  const gridLines = gridY.map(p => `<line x1="${padL}" y1="${y(p)}" x2="${W - padR}" y2="${y(p)}" stroke="#E8E8E8" stroke-width="1"/>`).join('');
  const yLabels = gridY.map(p => `<text x="${padL - 6}" y="${y(p) + 4}" text-anchor="end" fill="#A1A1AA" font-size="11">${p}%</text>`).join('');
  const xLabels = years.map((yr, i) => `<text x="${x(i)}" y="${H - 9}" text-anchor="middle" fill="#A1A1AA" font-size="11">${yr}</text>`).join('');
  const feeLabels = years.map((_, i) => {
    const f = scenarios.soft[i].fee;
    return f > 0 ? `<text x="${x(i)}" y="${H - 9 + 14}" text-anchor="middle" fill="#CCC" font-size="9">+$${f.toFixed(0)}</text>` : '';
  }).join('');

  function buildLayer(key) {
    const data = scenarios[key];
    const pts = data.map((r, i) => `${i === 0 ? 'M' : 'L'}${x(i)},${y(r.demand)}`).join('');
    const bot = y(0);
    const area = `M${x(0)},${bot}${pts.slice(1)}L${x(data.length - 1)},${bot}Z`;
    const dots = data.map((r, i) => `<circle cx="${x(i)}" cy="${y(r.demand)}" r="4" fill="${colors[key]}" stroke="#fff" stroke-width="2"/>`).join('');
    const last = data[data.length - 1];
    let endLabel;
    if (last.demand < 1) {
      endLabel = `<text x="${x(data.length - 1) - 8}" y="${y(last.demand) - 7}" text-anchor="end" fill="${colors[key]}" font-size="12" font-weight="700">&lt;1%</text>`;
    } else if (key === 'notax') {
      endLabel = `<text x="${x(data.length - 1) - 8}" y="${y(last.demand) - 6}" text-anchor="end" fill="${colors[key]}" font-size="11" font-weight="600">${last.demand}%</text>`;
    } else {
      endLabel = `<text x="${x(data.length - 1) + 8}" y="${y(last.demand) + 4}" fill="${colors[key]}" font-size="12" font-weight="700">${last.demand}%</text>`;
    }
    return { pts, area, dots, endLabel, color: colors[key] };
  }

  const layers = {};
  for (const k of ['notax', 'soft']) layers[k] = buildLayer(k);

  const svgContent = `<svg viewBox="0 0 ${W} ${H}" class="graph">
    ${gridLines}
    ${yLabels}
    ${xLabels}
    ${feeLabels}
    <path d="${layers.notax.area}" fill="${colors.notax}" opacity="0.05"/>
    <path d="${layers.soft.area}" fill="${colors.soft}" opacity="0.06"/>
    <path d="${layers.notax.pts}" fill="none" stroke="${colors.notax}" stroke-width="2" stroke-linejoin="round" stroke-dasharray="6,3"/>
    <path d="${layers.soft.pts}" fill="none" stroke="${colors.soft}" stroke-width="2.5" stroke-linejoin="round"/>
    ${layers.notax.dots}
    ${layers.soft.dots}
    ${layers.notax.endLabel}
    ${layers.soft.endLabel}
  </svg>`;
</script>

<section id="tax">
  <div class="hero-bg">
    <div class="hero-content">
      <span class="section-label">Action 1</span>
      <h2>
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="flex-shrink:0;margin-top:2px;">
          <path d="M22 2L11 13"/>
          <path d="M22 2l-7 20-4-9-9-4 20-7z"/>
        </svg>
        Polluters Pay
      </h2>
      <p class="lede">
        The Green Flight Plan levies fees on <strong>both</strong> of aviation's fuels — but not equally. Both fees rise every year so we create a clear and predictable long term incentive to move. It also gives a much clearer business case to doing what airlines already say they want to do.
      </p>
    </div>
  </div>

  <div class="fuel-blocks">
    <div class="fuel-block fuel-block--lead">
      <div class="fuel-block__header">
        <svg class="fuel-block__icon" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/>
          <line x1="12" y1="9" x2="12" y2="13"/>
          <line x1="12" y1="17" x2="12.01" y2="17"/>
        </svg>
        <div>
          <h3>Leaded Avgas 100LL</h3>
          <span class="fuel-block__badge fuel-block__badge--danger">Highly dangerous</span>
        </div>
      </div>
      <strong>$1/gal yearly rising fee</strong>
      <strong> Previous Plan: Wait until Ban in 2031 and hope it doesn't get extended.</strong>
      <ul class="fuel-block__list">
        <li>We phased out leaded gas for cars in 1996 because of the health dangers.</li>
        <li>No safe blood-lead level — children near GA airports show elevated levels <a href="https://pubmed.ncbi.nlm.nih.gov/36712926/#:~:text=We%20analyze%20over%2014%2C000%20blood%20lead%20samples,quantities%20of%20avgas%20sold%20at%20the%20airport">Study</a></li>
        <li>EPA formally found aircraft lead emissions endanger public health (2023) <a href="https://www.epa.gov/newsreleases/epa-determines-lead-emissions-aircraft-engines-cause-or-contribute-air-pollution">Link</a></li>
      </ul>
    </div>
    <div class="fuel-block fuel-block--jet">
      <div class="fuel-block__header">
        <svg class="fuel-block__icon" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M8.5 14.5A2.5 2.5 0 0 0 11 12c0-1.38-.5-2-1-3-1.072-2.143-.224-4.054 2-6 .5 2.5 2 4.9 4 6.5 2 1.6 3 3.5 3 5.5a7 7 0 1 1-14 0c0-1.153.433-2.294 1-3a2.5 2.5 0 0 0 2.5 2.5z"/>
        </svg>
        <div>
          <h3>Conventional Jet Fuel</h3>
          <span class="fuel-block__badge fuel-block__badge--harm">Also harmful</span>
        </div>
      </div>
      <strong>$0.25/gal yearly rising fee</strong>
      <strong>Previous Plan: Hope airlines 2050 net zero plans work without outside action.</strong>
      <ul class="fuel-block__list">
        <li>Aviation accounts for ~2.5% of global CO₂ emissions — and growing</li>
        <li>Fossil jet fuel has no price incentive to switch to SAF today</li>
        <li>Small excise funds electric charging infrastructure at airports</li>
      </ul>
    </div>
  </div>

  <div class="content">
    <div class="section-header">
      <h3>Scenario: what a rising fee could do</h3>
      <p>
        These are <strong>illustrative scenarios, not predictions</strong>. The actual outcome
        depends on how quickly unleaded alternatives reach the market.
      </p>
    </div>

    <div class="graph-card">
      <div class="graph-head">
        <h4>100LL demand under a rising fee</h4>
        <div class="graph-legend">
          <span class="legend-line soft"></span> Increasing Fee
          <span class="legend-line notax"></span> No Fee
        </div>
      </div>
      <div class="graph-wrap">{@html svgContent}</div>
    </div>

    <div class="scenario-labels">
      <div class="slabel">
        <span class="slabel-dot" style="background:{colors.soft}"></span>
        <strong>{scenarios.soft[0].demand}% &rarr; {scenarios.soft[scenarios.soft.length-1].demand}%</strong>
        <span>Soft Landing &mdash; UL94 dominates, gradual phase-out</span>
      </div>
      <div class="slabel">
        <span class="slabel-dot" style="background:{colors.notax}"></span>
        <strong>{scenarios.notax[0].demand}% &rarr; {scenarios.notax[scenarios.notax.length-1].demand}%</strong>
        <span>No Fee &mdash; Voluntary transition, demand barely moves</span>
      </div>
    </div>
  </div>
</section>

<style>
  section { display: flex; flex-direction: column; align-items: center; }

  .hero-bg {
    width: 100%;
    background: linear-gradient(135deg, #3B0D0D 0%, #7A1A1A 40%, #B94A4A 100%);
    color: #fff;
    display: flex;
    justify-content: center;
    padding: 3rem 1.25rem;
  }

  .hero-content { max-width: 720px; display: flex; flex-direction: column; gap: 0.5rem; }

  .section-label {
    font-size: 0.75rem; font-weight: 700; text-transform: uppercase;
    letter-spacing: 0.12em; opacity: 0.6;
  }

  .hero-content h2 {
    font-size: 2rem; font-weight: 600; letter-spacing: -0.02em;
    margin: 0; line-height: 1.15; display: flex; align-items: center; gap: 0.5rem;
  }

  .hero-content .lede {
    font-size: 1rem; line-height: 1.55; margin: 0; opacity: 0.9; max-width: 600px;
  }

  .hero-content .lede :global(strong) { color: #FFB3B3; }

  .content {
    max-width: 720px; display: flex; flex-direction: column; gap: 1.5rem; padding: 2rem 1.25rem;
  }

  .section-header h3 {
    font-size: 1.5rem; font-weight: 500; color: var(--neutral-900);
    margin: 0 0 0.5rem 0; letter-spacing: -0.02em;
  }

  .section-header p {
    font-size: 1rem; line-height: 1.6; color: var(--neutral-500); margin: 0;
  }

  .graph-card {
    border: 1px solid var(--neutral-200); border-radius: 10px; padding: 1rem; background: #fff;
  }

  .graph-head {
    display: flex; justify-content: space-between; align-items: center;
    margin-bottom: 1rem; flex-wrap: wrap; gap: 0.5rem;
  }

  .graph-head h4 { font-size: 1rem; font-weight: 600; color: var(--neutral-700); margin: 0; }

  .graph-legend {
    display: flex; align-items: center; gap: 1.25rem; font-size: 0.8125rem; color: var(--neutral-500);
  }

  .legend-line {
    display: inline-block; width: 18px; height: 3px; border-radius: 2px; margin-right: 4px; vertical-align: middle;
  }

  .legend-line.soft { background: #D94A4A; }
  .legend-line.notax { background: #999999; }

  .graph-wrap { width: 100%; }

  .scenario-labels {
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
  }

  .slabel {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    font-size: 0.875rem;
    color: var(--neutral-600);
  }

  .slabel-dot {
    width: 12px; height: 12px; border-radius: 50%; flex-shrink: 0;
  }

  .slabel strong {
    font-size: 0.9rem;
    color: var(--neutral-900);
    white-space: nowrap;
    min-width: 5em;
  }

  .fuel-blocks {
    display: flex;
    gap: 1.25rem;
    max-width: 720px;
    width: 100%;
    padding: 1.5rem 1.25rem;
    box-sizing: border-box;
  }

  .fuel-block {
    flex: 1;
    border-radius: 10px;
    padding: 1.25rem;
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .fuel-block--lead {
    background: #FFF5F0;
    border: 1px solid #FFD6C0;
  }

  .fuel-block--jet {
    background: #FFFDF5;
    border: 1px solid #F0E6C0;
  }

  .fuel-block__header {
    display: flex;
    gap: 0.75rem;
    align-items: flex-start;
  }

  .fuel-block__icon {
    flex-shrink: 0;
    margin-top: 2px;
  }

  .fuel-block--lead .fuel-block__icon {
    color: #B94A4A;
  }

  .fuel-block--jet .fuel-block__icon {
    color: #B98A2A;
  }

  .fuel-block__header h3 {
    font-size: 1rem;
    font-weight: 600;
    color: var(--neutral-900);
    margin: 0 0 0.25rem 0;
  }

  .fuel-block__badge {
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    padding: 2px 8px;
    border-radius: 4px;
    display: inline-block;
  }

  .fuel-block__badge--danger {
    background: #B94A4A;
    color: #fff;
  }

  .fuel-block__badge--harm {
    background: #D4A030;
    color: #fff;
  }

  .fuel-block__list {
    margin: 0;
    padding: 0;
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .fuel-block__list li {
    font-size: 0.875rem;
    line-height: 1.45;
    color: var(--neutral-600);
    padding-left: 1.25rem;
    position: relative;
  }

  .fuel-block__list li::before {
    content: "\00b7";
    position: absolute;
    left: 0.25rem;
    font-weight: 700;
    color: var(--neutral-400);
  }

  @media (max-width: 640px) {
    .fuel-blocks {
      flex-direction: column;
    }
  }
</style>
