<script>
  export let faqs = [];
</script>

<div class="faq-section">
  <div class="section-header"><h2>Questions fréquentes</h2></div>
  <div class="faq-grid">
    {#each faqs as faq}
      <div class="faq-item">
        <div class="faq-q">{faq.q}</div>
        <div class="faq-a">{@html faq.a}</div>
      </div>
    {/each}
  </div>
</div>

<style>
  .faq-section { max-width: 1200px; margin: 32px auto 0; padding: 0 32px; }
  .section-header { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 28px; gap: 16px; }
  .section-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.65rem; font-weight: 800; position: relative; }
  .section-header h2::after { content: ''; display: block; width: 40px; height: 4px; background: var(--orange); border-radius: 2px; margin-top: 8px; }

  .faq-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
  .faq-item { background: var(--white); border: 1px solid var(--border); border-radius: var(--r); padding: 22px 24px; box-shadow: var(--sh); }
  .faq-q { font-family: 'Montserrat', sans-serif; font-size: .95rem; font-weight: 700; margin-bottom: 10px; display: flex; align-items: flex-start; gap: 10px; }
  .faq-q::before { content: '?'; display: flex; align-items: center; justify-content: center; width: 22px; height: 22px; min-width: 22px; background: var(--orange); color: var(--dark); border-radius: 50%; font-size: .72rem; font-weight: 800; margin-top: 1px; }
  .faq-a { font-size: .875rem; color: var(--mid); line-height: 1.65; padding-left: 32px; }

  @media (max-width: 900px) {
    .faq-grid { grid-template-columns: 1fr; }
  }
  @media (max-width: 600px) {
    .faq-section { padding: 0 16px; }
  }
</style>
