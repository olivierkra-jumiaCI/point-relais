<div class="tarif-section">
  <div class="section-header"><h2>Tarification Transparente</h2></div>

  <div class="tarif-layout">
    <!-- Colonne gauche : tailles + zones -->
    <div class="tarif-left">
      <div class="block-card">
        <h3 class="block-title">Tailles de Colis</h3>
        <div class="size-list">
          <div class="size-item">
            <span class="size-badge">Petit</span>
            <div>
              <strong>Petit Colis (40×20×13 cm)</strong>
              <p>Téléphones, chemises, livres, etc.</p>
            </div>
          </div>
          <div class="size-item">
            <span class="size-badge">Moyen</span>
            <div>
              <strong>Colis Moyen (70×30×20 cm)</strong>
              <p>Mixeurs, chaussures, ordinateurs portables.</p>
            </div>
          </div>
          <div class="size-item">
            <span class="size-badge">Grand</span>
            <div>
              <strong>Grand Colis (100×100×62 cm)</strong>
              <p>Micro-ondes, articles volumineux.</p>
            </div>
          </div>
        </div>
      </div>

      <div class="block-card">
        <h3 class="block-title">Zones &amp; Localités</h3>
        <div class="zone-list">
          <div class="zone-tag z1">
            <span class="zone-label">Zone 1</span>
            <span class="zone-cities">Abidjan et sa périphérie immédiate</span>
          </div>
          <div class="zone-tag z2">
            <span class="zone-label">Zone 2</span>
            <span class="zone-cities">Agboville, Grand-Bassam, Dabou, Anyama…</span>
          </div>
          <div class="zone-tag z3">
            <span class="zone-label">Zone 3</span>
            <span class="zone-cities">Bouaké, Yamoussoukro, Daloa, San-Pédro…</span>
          </div>
          <div class="zone-tag z4">
            <span class="zone-label">Zone 4</span>
            <span class="zone-cities">Korhogo, Man, Bondoukou, Soubré…</span>
          </div>
          <div class="zone-tag z5">
            <span class="zone-label">Zone 5</span>
            <span class="zone-cities">Tabou, Bouna, Odienné, Tingréla…</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Colonne droite : trajets populaires -->
    <div class="tarif-right">
      <h3 class="block-title">Trajets Populaires <span class="subtitle">(Petit Colis)</span></h3>
      <table class="trajet-table">
        <thead>
          <tr>
            <th>TRAJET</th>
            <th>DÉLAI</th>
            <th>PRIX</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>
              <span class="city">Abidjan</span>
              <span class="zone-pill z1">Zone 1</span>
              <span class="arrow">→</span>
              <span class="city">Akoupé</span>
              <span class="zone-pill z1">Zone 1</span>
            </td>
            <td><span class="badge badge-green">J+2</span></td>
            <td><strong class="price">750 FCFA</strong></td>
          </tr>
          <tr>
            <td>
              <span class="city">Abidjan</span>
              <span class="zone-pill z1">Zone 1</span>
              <span class="arrow">→</span>
              <span class="city">Agboville</span>
              <span class="zone-pill z2">Zone 2</span>
            </td>
            <td><span class="badge badge-orange">J+3</span></td>
            <td><strong class="price">900 FCFA</strong></td>
          </tr>
          <tr>
            <td>
              <span class="city">Abidjan</span>
              <span class="zone-pill z1">Zone 1</span>
              <span class="arrow">→</span>
              <span class="city">Bouaké</span>
              <span class="zone-pill z3">Zone 3</span>
            </td>
            <td><span class="badge badge-grey">J+4</span></td>
            <td><strong class="price">1 000 FCFA</strong></td>
          </tr>
          <tr>
            <td>
              <span class="city">Abidjan</span>
              <span class="zone-pill z1">Zone 1</span>
              <span class="arrow">→</span>
              <span class="city">Korhogo</span>
              <span class="zone-pill z4">Zone 4</span>
            </td>
            <td><span class="badge badge-grey">J+5</span></td>
            <td><strong class="price">1 000 FCFA</strong></td>
          </tr>
          <tr>
            <td>
              <span class="city">Abidjan</span>
              <span class="zone-pill z1">Zone 1</span>
              <span class="arrow">→</span>
              <span class="city">Tabou</span>
              <span class="zone-pill z5">Zone 5</span>
            </td>
            <td><span class="badge badge-red">J+5</span></td>
            <td><strong class="price">1 000 FCFA</strong></td>
          </tr>
        </tbody>
      </table>
      <a href="https://ci.jumia.is/cms/1_2023/W42/FLR/tarifs-points-relais.pdf" target="_blank" class="tarif-link">
        Voir la grille tarifaire complète →
      </a>
    </div>
  </div>
</div>

<style>
  .tarif-section { max-width: 1200px; margin: 32px auto 0; padding: 0 32px; }
  .section-header { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 28px; gap: 16px; }
  .section-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.65rem; font-weight: 800; position: relative; }
  .section-header h2::after { content: ''; display: block; width: 40px; height: 4px; background: var(--orange); border-radius: 2px; margin-top: 8px; }

  .tarif-layout {
    display: grid;
    grid-template-columns: 340px 1fr;
    gap: 24px;
    align-items: start;
  }

  .block-card {
    background: var(--white);
    border: 1px solid var(--border);
    border-radius: var(--r-lg);
    padding: 24px;
    box-shadow: var(--sh);
    margin-bottom: 20px;
  }
  .block-card:last-child { margin-bottom: 0; }

  .block-title {
    font-family: 'Montserrat', sans-serif;
    font-size: .95rem;
    font-weight: 700;
    color: #1A1A1A;
    margin: 0 0 16px;
  }
  .block-title .subtitle {
    font-weight: 400;
    font-size: .85rem;
    color: var(--mid);
  }

  /* Size list */
  .size-list { display: flex; flex-direction: column; gap: 14px; }
  .size-item { display: flex; align-items: flex-start; gap: 14px; }
  .size-badge {
    background: var(--warm-grey);
    color: var(--mid);
    font-size: .7rem;
    font-weight: 700;
    border-radius: 8px;
    padding: 4px 10px;
    white-space: nowrap;
    margin-top: 2px;
  }
  .size-item strong { font-family: 'Montserrat', sans-serif; font-size: .82rem; color: #1A1A1A; display: block; }
  .size-item p { font-size: .78rem; color: var(--mid); margin: 2px 0 0; line-height: 1.4; }

  /* Zone list */
  .zone-list { display: flex; flex-direction: column; gap: 10px; }
  .zone-tag { display: flex; align-items: center; gap: 10px; }
  .zone-label {
    font-family: 'Montserrat', sans-serif;
    font-size: .73rem;
    font-weight: 700;
    border-radius: 20px;
    padding: 3px 10px;
    white-space: nowrap;
  }
  .zone-cities { font-size: .8rem; color: var(--mid); }

  .z1 .zone-label { background: #FFF3CD; color: #856404; }
  .z2 .zone-label { background: #D1ECF1; color: #0C5460; }
  .z3 .zone-label { background: #D4EDDA; color: #155724; }
  .z4 .zone-label { background: #E2D9F3; color: #4A235A; }
  .z5 .zone-label { background: #FDDDDD; color: #721C24; }

  /* Right side */
  .tarif-right {
    background: var(--white);
    border: 1px solid var(--border);
    border-radius: var(--r-lg);
    padding: 28px;
    box-shadow: var(--sh);
  }

  /* Table */
  .trajet-table { width: 100%; border-collapse: collapse; margin-top: 16px; }
  .trajet-table thead th {
    font-family: 'Montserrat', sans-serif;
    font-size: .72rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: .07em;
    color: var(--mid);
    padding: 8px 12px;
    border-bottom: 2px solid var(--border);
    text-align: left;
  }
  .trajet-table tbody tr { border-bottom: 1px solid var(--border); transition: background .15s; }
  .trajet-table tbody tr:last-child { border-bottom: none; }
  .trajet-table tbody tr:hover { background: var(--warm-grey); }
  .trajet-table td { padding: 14px 12px; vertical-align: middle; }
  .trajet-table td:first-child { font-size: .83rem; }

  .city { font-weight: 600; color: #1A1A1A; font-family: 'Montserrat', sans-serif; }
  .arrow { color: var(--orange); font-weight: 700; margin: 0 6px; }

  .zone-pill {
    font-size: .65rem;
    font-weight: 700;
    border-radius: 20px;
    padding: 2px 7px;
    margin-left: 4px;
    vertical-align: middle;
  }
  .z1.zone-pill { background: #FFF3CD; color: #856404; }
  .z2.zone-pill { background: #D1ECF1; color: #0C5460; }
  .z3.zone-pill { background: #D4EDDA; color: #155724; }
  .z4.zone-pill { background: #E2D9F3; color: #4A235A; }
  .z5.zone-pill { background: #FDDDDD; color: #721C24; }

  .badge {
    display: inline-flex; align-items: center;
    border-radius: 50px; padding: 3px 10px;
    font-size: .75rem; font-weight: 700;
    white-space: nowrap;
  }
  .badge-green  { background: #D4EFDF; color: #1E7E45; }
  .badge-orange { background: var(--orange-lt); color: var(--orange-dk); }
  .badge-grey   { background: var(--warm-grey); color: var(--mid); }
  .badge-red    { background: #FDDDDD; color: var(--red); }

  .price { font-family: 'Montserrat', sans-serif; font-size: .9rem; color: var(--orange); white-space: nowrap; }

  .tarif-link {
    display: inline-block;
    margin-top: 20px;
    color: var(--orange);
    font-family: 'Montserrat', sans-serif;
    font-size: .82rem;
    font-weight: 700;
    text-decoration: none;
  }
  .tarif-link:hover { text-decoration: underline; }

  @media (max-width: 900px) {
    .tarif-layout { grid-template-columns: 1fr; }
    .tarif-section { padding: 0 16px; }
  }
</style>
