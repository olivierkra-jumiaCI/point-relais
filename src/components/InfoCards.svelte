<div class="info-grid-section">
  <div class="section-header" style="margin-top:0;"><h2>Informations pratiques</h2></div>
  <div class="info-3col">
    <div class="icard orange">
      <span class="icard-icon">🕗</span>
      <h3>Horaires d'ouverture</h3>
      <p>Toutes nos agences sont ouvertes :</p><br/>
      <span class="big-val">08h – 18h</span>
      <span class="val-label">Du Lundi au Samedi</span><br/><br/>
      <p>Fermées les dimanches et jours fériés nationaux.</p>
    </div>
    <div class="icard green">
      <span class="icard-icon">📞</span>
      <h3>Nous contacter</h3>
      <p>Une question sur votre retrait ou retour ?</p><br/>
      <span class="big-val">25 20 00 61 61</span>
      <span class="val-label">Centre d'appel Jumia CI</span><br/><br/>
      <p>Disponible du Lundi au Samedi de 8h à 18h.</p>
    </div>
    <div class="icard red">
      <span class="icard-icon">📦</span>
      <h3>Types de colis acceptés</h3>
      <ul>
        <li>Colis standard acceptés partout</li>
        <li>Grands colis selon disponibilité</li>
        <li>🔴 Indicates grandes colis non acceptés</li>
        <li>↩ Indique le retour autorisé</li>
      </ul>
    </div>
  </div>
</div>

<style>
  .info-grid-section { max-width: 1200px; margin: 52px auto 0; padding: 0 32px; }
  .section-header { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 28px; gap: 16px; }
  .section-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.65rem; font-weight: 800; position: relative; }
  .section-header h2::after { content: ''; display: block; width: 40px; height: 4px; background: var(--orange); border-radius: 2px; margin-top: 8px; }

  .info-3col { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
  .icard { background: var(--white); border: 1px solid var(--border); border-radius: var(--r-lg); padding: 28px; box-shadow: var(--sh); position: relative; overflow: hidden; }
  .icard::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 4px; border-radius: var(--r-lg) var(--r-lg) 0 0; }
  .icard.orange::before { background: var(--orange); }
  .icard.green::before { background: var(--green); }
  .icard.red::before { background: var(--red); }
  .icard-icon { font-size: 2rem; margin-bottom: 14px; display: block; }
  .icard h3 { font-family: 'Montserrat', sans-serif; font-size: 1rem; font-weight: 700; margin-bottom: 10px; }
  .icard p, .icard li { font-size: .875rem; color: var(--mid); line-height: 1.65; }
  .icard ul { padding-left: 16px; }
  .icard li { margin-bottom: 4px; }
  .icard .big-val { font-family: 'Montserrat', sans-serif; font-size: 1.6rem; font-weight: 800; color: var(--dark); display: block; margin-bottom: 4px; }
  .icard .val-label { font-size: .78rem; color: var(--mid); text-transform: uppercase; letter-spacing: .06em; font-weight: 600; }

  @media (max-width: 900px) {
    .info-3col { grid-template-columns: 1fr 1fr; }
  }
  @media (max-width: 600px) {
    .info-3col { grid-template-columns: 1fr; }
    .info-grid-section { padding: 0 16px; }
  }
</style>
