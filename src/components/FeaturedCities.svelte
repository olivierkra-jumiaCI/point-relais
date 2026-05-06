<script>
  export let cities = [];
</script>

<div class="section anim-3">
  <div class="section-header">
    <h2>Nouvelles ouvertures</h2>
    <a href="#finder" class="section-link">Voir toutes les villes →</a>
  </div>
  <div class="cities-grid">
    {#each cities as city}
      <a class="city-card" href={city.link} target="_blank">
        <div class="city-card-overlay">Nouveau</div>
        <img class="city-card-img" src={city.img} alt={city.name}/>
        <div class="city-card-body">
          <div class="city-card-name">{city.name}</div>
          <span class="city-card-tag">📍 Région de {city.region}</span>
        </div>
      </a>
    {/each}
  </div>
</div>

<style>
  .section { max-width: 1200px; margin: 52px auto 0; padding: 0 32px; }
  .section-header { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 28px; gap: 16px; }
  .section-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.65rem; font-weight: 800; position: relative; }
  .section-header h2::after { content: ''; display: block; width: 40px; height: 4px; background: var(--orange); border-radius: 2px; margin-top: 8px; }
  .section-link { font-size: .85rem; font-weight: 600; color: var(--orange-dk); text-decoration: none; display: flex; align-items: center; gap: 4px; white-space: nowrap; }
  .section-link:hover { text-decoration: underline; }

  .cities-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
  .city-card { border-radius: var(--r-lg); overflow: hidden; box-shadow: var(--sh); position: relative; cursor: pointer; transition: transform .25s, box-shadow .25s; display: block; text-decoration: none; }
  .city-card:hover { transform: translateY(-4px); box-shadow: var(--sh-lg); }
  .city-card-img { width: 100%; height: 200px; object-fit: cover; display: block; background: var(--border); }
  .city-card-body { background: var(--white); padding: 18px 20px; border: 1px solid var(--border); border-top: none; border-radius: 0 0 var(--r-lg) var(--r-lg); }
  .city-card-name { font-family: 'Montserrat', sans-serif; font-size: 1.05rem; font-weight: 700; margin-bottom: 6px; color: var(--dark); }
  .city-card-tag { display: inline-flex; align-items: center; gap: 5px; font-size: .76rem; font-weight: 500; color: var(--mid); background: var(--warm-grey); border-radius: 50px; padding: 3px 10px; }
  .city-card-overlay { position: absolute; top: 12px; right: 12px; background: var(--orange); color: var(--dark); font-family: 'Montserrat', sans-serif; font-size: .72rem; font-weight: 700; border-radius: 50px; padding: 4px 10px; text-transform: uppercase; letter-spacing: .05em; }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(28px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .anim-3 { animation: fadeUp .6s .3s ease both; }

  @media (max-width: 900px) {
    .cities-grid { grid-template-columns: repeat(2, 1fr); }
  }
  @media (max-width: 600px) {
    .cities-grid { grid-template-columns: 1fr; }
    .section { padding: 0 16px; }
  }
</style>
