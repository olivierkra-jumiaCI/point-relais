<script>
  import { onMount } from 'svelte';
  import L from 'leaflet';
  import 'leaflet/dist/leaflet.css';

  export let agencies = [];
  export let regions = [];

  let map;
  let markers;
  let searchQuery = '';
  let orangeIcon;

  function flyTo(lat, lng, zoom) {
    if (map) {
      map.flyTo([lat, lng], zoom, { duration: 1.2 });
    }
  }

  $: filteredRegions = searchQuery 
    ? regions.filter(r => r.name.toLowerCase().includes(searchQuery.toLowerCase()))
    : regions;

  async function fetchSheetData() {
    try {
      const response = await fetch('https://docs.google.com/spreadsheets/d/1M52gDOvkoXZtCA7RSmHM1vy4ksO6H5fdQQ-twAkRqKk/export?format=csv&gid=0');
      const text = await response.text();
      
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
            .bindPopup(`<div style="font-family:'Montserrat',sans-serif;min-width:180px;padding:8px"><strong style="color:#FF9900;display:block;margin-bottom:6px;font-size:14px">${a.n}</strong><p style="color:#444;line-height:1.4;margin:0;font-size:12px">${a.address || 'Point Relais Jumia'}</p></div>`);
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
    map = L.map('jumiaMap', { 
      zoomControl: false,
      attributionControl: false 
    }).setView([7.0, -5.5], 6);

    L.control.zoom({ position: 'topright' }).addTo(map);

    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
      subdomains: 'abcd', maxZoom: 19
    }).addTo(map);

    orangeIcon = L.divIcon({
      className: 'custom-pin',
      html: '<div class="pin-wrap"><div class="pin-inner"></div></div>',
      iconSize: [28, 28], iconAnchor: [14, 28], popupAnchor: [0, -28]
    });

    markers = L.featureGroup().addTo(map);
    fetchSheetData();

    // Ensure map resizes correctly
    setTimeout(() => {
      map.invalidateSize();
    }, 200);
  });
</script>

<div class="map-finder-section" id="finder">
  <div class="map-container">
    <div class="map-sidebar">
      <div class="search-box">
        <div class="search-input-wrap">
          <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          </svg>
          <input 
            bind:value={searchQuery} 
            type="text" 
            placeholder="Chercher une ville ou une localité..."
          />
        </div>
      </div>

      <div class="region-list-container">
        {#each filteredRegions as region}
          <button class="region-item" on:click={() => flyTo(region.lat, region.lng, region.zoom)}>
            <span class="region-name">{region.name}</span>
            <span class="plus-icon">+</span>
          </button>
        {/each}
      </div>
    </div>

    <div class="map-canvas-wrap">
      <div id="jumiaMap"></div>
    </div>
  </div>
</div>

<style>
  .map-finder-section {
    max-width: 1240px;
    margin: 80px auto;
    padding: 0 20px;
    width: 100%;
  }

  .map-container {
    display: flex;
    background: #F3EFE6;
    border-radius: 32px;
    overflow: hidden;
    box-shadow: 0 30px 90px rgba(0,0,0,0.08);
    height: 700px;
    border: 1px solid rgba(0,0,0,0.05);
    width: 100%;
  }

  /* Sidebar */
  .map-sidebar {
    width: 380px;
    min-width: 380px;
    display: flex;
    flex-direction: column;
    background: #F3EFE6;
    border-right: 1px solid rgba(0,0,0,0.06);
    z-index: 10;
  }

  .search-box {
    padding: 24px 20px;
  }

  .search-input-wrap {
    position: relative;
    display: flex;
    align-items: center;
  }

  .search-icon {
    position: absolute;
    left: 18px;
    width: 18px;
    height: 18px;
    color: #4A90E2;
  }

  .search-box input {
    width: 100%;
    height: 52px;
    background: rgba(0,0,0,0.05);
    border: none;
    border-radius: 26px;
    padding: 0 20px 0 50px;
    font-family: 'Montserrat', sans-serif;
    font-size: 14px;
    font-weight: 500;
    color: #444;
    outline: none;
    transition: background 0.2s;
  }

  .search-box input:focus {
    background: rgba(0,0,0,0.08);
  }

  .region-list-container {
    flex: 1;
    overflow-y: auto;
    padding: 0 0 20px;
  }

  .region-list-container::-webkit-scrollbar {
    width: 6px;
  }

  .region-list-container::-webkit-scrollbar-thumb {
    background: rgba(0,0,0,0.1);
    border-radius: 10px;
  }

  .region-item {
    width: 100%;
    background: none;
    border: none;
    border-bottom: 1px solid rgba(0,0,0,0.03);
    padding: 20px 24px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    cursor: pointer;
    text-align: left;
    transition: background 0.2s;
  }

  .region-item:hover {
    background: rgba(255,255,255,0.4);
  }

  .region-name {
    font-family: 'Montserrat', sans-serif;
    font-size: 14.5px;
    font-weight: 700;
    color: #1A1A1A;
    line-height: 1.4;
  }

  .plus-icon {
    font-size: 20px;
    color: #E0D7C6;
    font-weight: 300;
    margin-left: 12px;
  }

  /* Map Canvas */
  .map-canvas-wrap {
    flex: 1;
    background: #f8f8f8;
    position: relative;
  }

  #jumiaMap {
    width: 100%;
    height: 100%;
  }

  /* Custom Leaflet Pin */
  :global(.custom-pin) {
    background: none !important;
    border: none !important;
  }

  :global(.pin-wrap) {
    width: 28px;
    height: 28px;
    background: #FF9900;
    border-radius: 50% 50% 50% 0;
    transform: rotate(-45deg);
    border: 3px solid #FFF;
    box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  :global(.pin-inner) {
    width: 8px;
    height: 8px;
    background: #FFF;
    border-radius: 50%;
  }

  @media (max-width: 900px) {
    .map-container {
      flex-direction: column;
      height: auto;
      border-radius: 24px;
    }

    .map-sidebar {
      width: 100%;
      min-width: 100%;
      height: 400px;
      border-right: none;
      border-bottom: 1px solid rgba(0,0,0,0.05);
    }

    .map-canvas-wrap {
      height: 450px;
    }
  }
</style>
