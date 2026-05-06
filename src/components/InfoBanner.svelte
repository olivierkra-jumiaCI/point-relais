<div class="info-banner anim-2">
  <div class="info-card">
    <div class="info-icon">🔄</div>
    <div class="info-text">
      <h3>Retrait & Retour en point relais</h3>
      <p>Choisissez une agence lors de votre passation de commande. Les agences autorisant le retour sont identifiées par le logo <span class="return-badge">↩ Retour autorisé</span>. Toutes nos agences sont ouvertes <strong>du Lundi au Samedi de 08h00 à 18h00</strong>.</p>
    </div>
  </div>
</div>

<style>
  .info-banner { max-width: 1200px; margin: 40px auto 0; padding: 0 32px; }
  .info-card { background: var(--white); border: 1px solid var(--border); border-radius: var(--r-lg); padding: 28px 32px; display: flex; align-items: center; gap: 24px; box-shadow: var(--sh); }
  .info-icon { width: 56px; height: 56px; border-radius: var(--r); background: var(--orange-lt); display: flex; align-items: center; justify-content: center; flex-shrink: 0; font-size: 1.6rem; }
  .info-text h3 { font-family: 'Montserrat', sans-serif; font-size: 1rem; font-weight: 700; margin-bottom: 6px; }
  .info-text p { font-size: .9rem; color: var(--mid); line-height: 1.6; }
  .return-badge { display: inline-flex; align-items: center; gap: 6px; background: var(--orange-lt); border: 1px solid var(--orange-mid); border-radius: 6px; padding: 3px 10px; font-size: .78rem; font-weight: 600; color: var(--orange-dk); margin-left: 8px; vertical-align: middle; }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(28px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .anim-2 { animation: fadeUp .6s .15s ease both; }

  @media (max-width: 600px) {
    .info-banner { padding: 0 16px; }
  }
</style>
