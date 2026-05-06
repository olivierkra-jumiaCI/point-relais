<header class="hero">
  <div class="hero-bg"></div>
  <div class="hero-grid"></div>
  <div class="hero-inner">
    <div class="hero-text">
      <div class="hero-badge anim-1"><span class="dot"></span>Réseau actif en Côte d'Ivoire</div>
      <h1 class="anim-2">Vos <span>agences & points relais</span> Jumia</h1>
      <p class="hero-sub anim-3">Retirez ou retournez votre commande dans plus de <strong style="color:var(--orange)">150 agences</strong> réparties dans plus de 100 villes à travers toute la Côte d'Ivoire. Rapide, simple et sans frais supplémentaires.</p>
    </div>
  </div>
</header>

<style>
  .hero { background: var(--dark); position: relative; overflow: hidden; }
  .hero-bg { position: absolute; inset: 0; background: radial-gradient(circle at 80% 20%, rgba(255, 153, 0, .18) 0%, transparent 50%), radial-gradient(circle at 10% 80%, rgba(255, 153, 0, .10) 0%, transparent 40%); pointer-events: none; }
  .hero-grid { position: absolute; inset: 0; background-image: linear-gradient(rgba(255, 153, 0, .04) 1px, transparent 1px), linear-gradient(90deg, rgba(255, 153, 0, .04) 1px, transparent 1px); background-size: 40px 40px; pointer-events: none; }
  .hero-inner { position: relative; max-width: 1200px; margin: auto; padding: 64px 32px 56px; display: flex; align-items: center; gap: 48px; }
  .hero-text { flex: 1; }
  .hero-badge { display: inline-flex; align-items: center; gap: 8px; background: rgba(255, 153, 0, .15); border: 1px solid rgba(255, 153, 0, .3); border-radius: 50px; padding: 6px 16px; font-family: 'Montserrat', sans-serif; font-size: .78rem; font-weight: 600; color: var(--orange); letter-spacing: .08em; text-transform: uppercase; margin-bottom: 20px; }
  .dot { width: 7px; height: 7px; border-radius: 50%; background: var(--orange); animation: pulse 2s infinite; }
  .hero h1 { font-family: 'Montserrat', sans-serif; font-size: clamp(2rem, 5vw, 3.4rem); font-weight: 800; color: var(--white); line-height: 1.1; margin-bottom: 16px; }
  .hero h1 span { color: var(--orange); }
  .hero-sub { font-size: 1.05rem; color: rgba(255, 255, 255, .65); line-height: 1.7; max-width: 520px; margin-bottom: 32px; }

  @keyframes pulse {
    0%, 100% { box-shadow: 0 0 0 0 rgba(255, 153, 0, .4); }
    50% { box-shadow: 0 0 0 10px rgba(255, 153, 0, 0); }
  }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(28px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .anim-1 { animation: fadeUp .6s ease both; }
  .anim-2 { animation: fadeUp .6s .15s ease both; }
  .anim-3 { animation: fadeUp .6s .3s ease both; }

  @media (max-width: 600px) {
    .hero-inner { flex-direction: column; padding: 36px 20px 32px; }
  }
</style>
