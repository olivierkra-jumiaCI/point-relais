<div class="zones-section">
  <div class="section-header"><h2>Délais de livraison estimés</h2></div>
  <div class="zones-table-wrap">
    <table class="zones-table">
      <thead>
        <tr>
          <th>Zone</th>
          <th>Villes concernées</th>
          <th>Délai standard</th>
          <th>Délai Express</th>
          <th>Retour</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Abidjan</td>
          <td>Treichville, Abobo, Adjamé, Cocody, Yopougon, Bingerville, Marcory, Koumassi, Plateau, Port-Bouet.</td>
          <td><span class="badge badge-orange">2 – 3 jours</span></td>
          <td><span class="badge badge-green">1 jour</span></td>
          <td><span class="badge badge-green">✓ Oui</span></td>
        </tr>
        <tr>
          <td>Grandes villes</td>
          <td>Aboisso, Agboville, Anyama, Azaguié, Bouaké, Dabou, Grand-Bassam, N'Douci, Sikensi, Tiassalé, Yamoussoukro.</td>
          <td><span class="badge badge-grey">3 – 4 jours</span></td>
          <td><span class="badge badge-orange">1 – 2 jours</span></td>
          <td><span class="badge badge-green">✓ Oui</span></td>
        </tr>
        <tr>
          <td>Villes secondaires</td>
          <td>Abengourou, Daloa, Korhogo, San-Pédro, Man, Divo, Gagnoa, Bondoukou, Soubré, Ferké, Issia, Katiola...</td>
          <td><span class="badge badge-grey">4 – 5 jours</span></td>
          <td><span class="badge badge-grey">2 – 3 jours</span></td>
          <td><span class="badge badge-green">✓ Oui</span></td>
        </tr>
        <tr>
          <td>Zones rurales</td>
          <td>Tabou, Grand-Béréby, Touba, Bouna, Tingrela, Guiglo, Danané, Odienné, Boundiali, Adiaké...</td>
          <td><span class="badge badge-red">4 – 6 jours</span></td>
          <td><span class="badge badge-grey">4 – 5 jours</span></td>
          <td><span class="badge badge-orange">Variable</span></td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

<style>
  .zones-section { max-width: 1200px; margin: 32px auto 0; padding: 0 32px; }
  .section-header { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 28px; gap: 16px; }
  .section-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.65rem; font-weight: 800; position: relative; }
  .section-header h2::after { content: ''; display: block; width: 40px; height: 4px; background: var(--orange); border-radius: 2px; margin-top: 8px; }

  .zones-table-wrap { background: var(--white); border: 1px solid var(--border); border-radius: var(--r-lg); overflow-x: auto; box-shadow: var(--sh); }
  .zones-table { width: 100%; border-collapse: collapse; }
  .zones-table thead th { background: var(--dark); color: var(--white); font-family: 'Montserrat', sans-serif; font-size: .8rem; font-weight: 700; text-transform: uppercase; letter-spacing: .07em; padding: 16px 20px; text-align: left; }
  .zones-table tbody tr { border-bottom: 1px solid var(--border); transition: background .15s; }
  .zones-table tbody tr:last-child { border-bottom: none; }
  .zones-table tbody tr:hover { background: var(--warm-grey); }
  .zones-table td { padding: 14px 20px; font-size: .875rem; }
  .zones-table td:first-child { font-weight: 600; }
  
  .badge { display: inline-flex; align-items: center; gap: 4px; border-radius: 50px; padding: 3px 10px; font-size: .75rem; font-weight: 600; }
  .badge-orange { background: var(--orange-lt); color: var(--orange-dk); }
  .badge-green { background: #D4EFDF; color: #1E7E45; }
  .badge-red { background: #FDDDDD; color: var(--red); }
  .badge-grey { background: var(--warm-grey); color: var(--mid); }

  @media (max-width: 600px) {
    .zones-section { padding: 0 16px; }
    .zones-table td { padding: 10px 12px; font-size: .75rem; }
  }
</style>
