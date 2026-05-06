<script>
  import { onMount } from 'svelte';

  export let agencies = [];
  export let regions = [];
  export let cityFilterCoords = {};

  let map;
  let markers;
  let searchQuery = '';
  let orangeIcon;

  let activeFilter = 'all';

  function flyTo(lat, lng, zoom) {
    if (map) {
      map.flyTo([lat, lng], zoom, { duration: 1.2 });
    }
  }

  function handleFilterClick(filter) {
    activeFilter = filter;
    const c = cityFilterCoords[filter] || cityFilterCoords.all;
    flyTo(c[0], c[1], c[2]);
  }

  $: filteredRegions = searchQuery 
    ? regions.filter(r => r.name.toLowerCase().includes(searchQuery.toLowerCase()))
    : regions;

  async function fetchSheetData() {
    try {
      const response = await fetch('https://docs.google.com/spreadsheets/d/1M52gDOvkoXZtCA7RSmHM1vy4ksO6H5fdQQ-twAkRqKk/export?format=csv&gid=0');
      const text = await response.text();
      
      // Basic CSV parser that handles simple quoting
      const rows = text.split('\r\n').map(row => {
        const result = [];
        let current = '';
        let inQuotes = false;
        for (let i = 0; i < row.length; i++) {
          const char = row[i];
          if (char === '"') inQuotes = !inQuotes;
          else if (char === ',' && !inQuotes) {
            result.push(current);
            current = '';
          } else {
            current += char;
          }
        }
        result.push(current);
        return result;
      });

      if (rows.length < 2) return;
      const headers = rows[0].map(h => h.trim());
      
      const pusIdx = headers.indexOf('pus');
      const longlatIdx = headers.indexOf('longlat');
      const statusIdx = headers.indexOf('statut');
      const addressIdx = headers.indexOf('adresse');

      agencies = rows.slice(1)
        .filter(row => row[statusIdx] === 'Live')
        .map(row => {
          const [lat, lng] = (row[longlatIdx] || '').split(',').map(Number);
          return {
            n: row[pusIdx],
            lat,
            lng,
            address: row[addressIdx]
          };
        })
        .filter(a => !isNaN(a.lat) && !isNaN(a.lng));

      if (markers && map) {
        markers.clearLayers();
        agencies.forEach(a => {
          const m = L.marker([a.lat, a.lng], { icon: orangeIcon })
            .bindPopup(`<div style="font-family:Roboto,sans-serif;min-width:180px"><strong style="color:#FF9900;font-family:Montserrat,sans-serif;display:block;margin-bottom:4px">${a.n}</strong><small style="color:#666;line-height:1.4;display:block">${a.address || 'Point Relais Jumia'}</small></div>`);
          markers.addLayer(m);
        });
        if (agencies.length > 0) {
          map.fitBounds(markers.getBounds().pad(0.1));
        }
      }
    } catch (e) {
      console.error('Error fetching sheet data:', e);
    }
  }

  onMount(() => {
    if (typeof L === 'undefined') return;

    map = L.map('jumiaMap', { zoomControl: true }).setView([7.0, -5.5], 6);

    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/">CARTO</a>',
      subdomains: 'abcd', maxZoom: 19
    }).addTo(map);

    orangeIcon = L.divIcon({
      className: '',
      html: '<div style="width:22px;height:22px;border-radius:50% 50% 50% 0;background:#FF9900;transform:rotate(-45deg);border:3px solid #fff;box-shadow:0 2px 6px rgba(0,0,0,.3)"><div style="width:7px;height:7px;background:#fff;border-radius:50%;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%)"></div></div>',
      iconSize: [22, 22], iconAnchor: [11, 22], popupAnchor: [0, -24]
    });

    markers = L.featureGroup().addTo(map);
    
    fetchSheetData();
  });
</script>

<div class="map-finder-section" id="finder">
  <div class="map-finder-header">
    <h2>Nos Points Relais &amp; Zones d'Expédition</h2>
    <p>Plus de 150 Points Relais, déposer un colis est aussi simple que de marcher dans la rue.</p>
    <div class="city-filters">
      {#each Object.keys(cityFilterCoords) as filter}
        <button 
          class="city-filter {activeFilter === filter ? 'active' : ''}" 
          on:click={() => handleFilterClick(filter)}
        >
          {#if filter === 'all'}Toutes les villes
          {:else if filter === 'abidjan'}Abidjan
          {:else if filter === 'yamoussoukro'}Yamoussoukro
          {:else if filter === 'bouake'}Bouaké
          {:else if filter === 'sanpedro'}San Pedro
          {:else if filter === 'other'}+100 Autres Villes
          {/if}
        </button>
      {/each}
    </div>
  </div>

  <div class="map-finder-body">
    <div class="map-left-panel">
      <div class="map-search-wrap">
        <span class="map-search-icon">🔍</span>
        <input bind:value={searchQuery} type="search" placeholder="Chercher une ville ou une localité..."/>
      </div>

      <div class="region-list">
        {#each filteredRegions as region}
          <div class="region-item">
            <button class="region-btn" on:click={() => flyTo(region.lat, region.lng, region.zoom)}>
              <span>{region.name}</span>
              <span class="region-plus">+</span>
            </button>
          </div>
        {/each}
      </div>
    </div>

    <div class="map-right-panel">
      <div id="jumiaMap"></div>
    </div>
  </div>
</div>

<style>
  .map-finder-section { max-width: 1200px; margin: 52px auto 0; padding: 0 32px; }
  .map-finder-header { text-align: center; margin-bottom: 32px; }
  .map-finder-header h2 { font-family: 'Montserrat', sans-serif; font-size: 1.9rem; font-weight: 800; margin-bottom: 10px; }
  .map-finder-header p { font-size: .95rem; color: var(--mid); margin-bottom: 24px; }
  .city-filters { display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; }
  .city-filter { background: var(--white); border: 1px solid var(--border); border-radius: 50px; padding: 8px 20px; font-family: 'Montserrat', sans-serif; font-size: .82rem; font-weight: 600; color: var(--mid); cursor: pointer; transition: all .2s; }
  .city-filter:hover { border-color: var(--orange); color: var(--orange-dk); }
  .city-filter.active { background: var(--orange); border-color: var(--orange); color: var(--dark); }
  .map-finder-body { display: flex; gap: 0; border-radius: var(--r-lg); overflow: hidden; box-shadow: var(--sh-lg); border: 1px solid var(--border); height: 580px; }
  .map-left-panel { width: 340px; min-width: 300px; background: var(--white); display: flex; flex-direction: column; flex-shrink: 0; border-right: 1px solid var(--border); }
  .map-search-wrap { position: relative; padding: 16px 16px 12px; }
  .map-search-icon { position: absolute; left: 28px; top: 50%; transform: translateY(-50%); color: var(--mid); font-size: .9rem; pointer-events: none; }
  
  input[type="search"] { width: 100%; background: var(--warm-grey); border: 1px solid var(--border); border-radius: 50px; padding: 10px 16px 10px 36px; font-family: 'Roboto', sans-serif; font-size: .85rem; color: var(--dark); outline: none; transition: border-color .2s; }
  input[type="search"]:focus { border-color: var(--orange); }

  .region-list { flex: 1; overflow-y: auto; padding: 0 0 8px; }
  .region-list::-webkit-scrollbar { width: 4px; }
  .region-list::-webkit-scrollbar-thumb { background: var(--border); border-radius: 4px; }
  .region-item { border-bottom: 1px solid var(--border); }
  .region-btn { width: 100%; background: none; border: none; padding: 14px 20px; display: flex; align-items: center; justify-content: space-between; gap: 12px; cursor: pointer; text-align: left; font-family: 'Roboto', sans-serif; font-size: .84rem; font-weight: 500; color: var(--dark); transition: background .15s, color .15s; }
  .region-btn:hover { background: var(--orange-lt); color: var(--orange-dk); }
  .region-plus { font-size: 1.1rem; color: var(--border); flex-shrink: 0; transition: color .15s; }
  .map-right-panel { flex: 1; min-width: 0; }
  #jumiaMap { width: 100%; height: 100%; }

  @media (max-width: 800px) {
    .map-finder-body { flex-direction: column; height: auto; }
    .map-left-panel { width: 100%; border-right: none; border-bottom: 1px solid var(--border); max-height: 260px; }
    #jumiaMap { height: 380px; }
    .map-finder-section { padding: 0 16px; }
  }
</style>
