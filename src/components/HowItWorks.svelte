<script>
  export let steps = [];
</script>

<div class="section" id="comment">
  <div class="section-header"><h2>Comment ça marche ?</h2></div>
  <div class="steps-grid">
    {#each steps as step}
      <div class="step-card">
        <div class="step-num">{step.num}</div>
        <span class="step-icon">{step.icon}</span>
        <h4>{step.title}</h4>
        <p>{step.text}</p>
      </div>
    {/each}
  </div>
</div>

<style>
  .section { max-width: 1200px; margin: 52px auto 0; padding: 0 32px; }
  .section-header { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 28px; gap: 16px; }
  .section-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.65rem; font-weight: 800; position: relative; }
  .section-header h2::after { content: ''; display: block; width: 40px; height: 4px; background: var(--orange); border-radius: 2px; margin-top: 8px; }

  .steps-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; position: relative; }
  .steps-grid::before { content: ''; position: absolute; top: 36px; left: calc(12.5% + 8px); right: calc(12.5% + 8px); height: 2px; background: linear-gradient(90deg, var(--orange), var(--orange-mid), var(--orange)); opacity: .35; pointer-events: none; }
  .step-card { background: var(--white); border: 1px solid var(--border); border-radius: var(--r-lg); padding: 28px 20px 24px; text-align: center; box-shadow: var(--sh); transition: transform .2s, box-shadow .2s; }
  .step-card:hover { transform: translateY(-3px); box-shadow: var(--sh-lg); }
  .step-num { width: 48px; height: 48px; border-radius: 50%; background: var(--orange-lt); border: 2px solid var(--orange-mid); display: flex; align-items: center; justify-content: center; font-family: 'Montserrat', sans-serif; font-size: 1.1rem; font-weight: 800; color: var(--orange-dk); margin: 0 auto 16px; position: relative; z-index: 1; }
  .step-icon { font-size: 1.5rem; display: block; margin-bottom: 12px; }
  .step-card h4 { font-family: 'Montserrat', sans-serif; font-size: .95rem; font-weight: 700; margin-bottom: 8px; }
  .step-card p { font-size: .82rem; color: var(--mid); line-height: 1.55; }

  @media (max-width: 900px) {
    .steps-grid { grid-template-columns: repeat(2, 1fr); }
    .steps-grid::before { display: none; }
  }
  @media (max-width: 600px) {
    .steps-grid { grid-template-columns: 1fr; }
    .section { padding: 0 16px; }
  }
</style>
