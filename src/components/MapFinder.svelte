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
  let expandedRegion = null;
  let markerMap = new Map();

  function toggleRegion(regionName, lat, lng, zoom) {
    if (expandedRegion === regionName) {
      expandedRegion = null;
    } else {
      expandedRegion = regionName;
      flyTo(lat, lng, zoom);
    }
  }

  function flyTo(lat, lng, zoom) {
    if (map) {
      map.flyTo([lat, lng], zoom, { duration: 1.2 });
    }
  }

  function handleAgencyClick(agency) {
    if (map) {
      map.flyTo([agency.lat, agency.lng], 15, { duration: 1 });
      const marker = markerMap.get(agency.n);
      if (marker) {
        marker.openPopup();
      }
    }
  }

  $: filteredAgencies = agencies.filter(a => 
    a.n.toLowerCase().includes(searchQuery.toLowerCase()) ||
    (a.address || '').toLowerCase().includes(searchQuery.toLowerCase()) ||
    (a.region || '').toLowerCase().includes(searchQuery.toLowerCase())
  );

  $: filteredRegions = regions.map(region => {
    const agenciesInRegion = filteredAgencies.filter(a => a.region === region.name);
    return {
      ...region,
      agencies: agenciesInRegion,
      visible: agenciesInRegion.length > 0 || region.name.toLowerCase().includes(searchQuery.toLowerCase())
    };
  }).filter(r => r.visible);

  // Auto-expand if searching
  $: if (searchQuery.length > 0 && filteredRegions.length > 0) {
    if (!expandedRegion || !filteredRegions.find(r => r.name === expandedRegion)) {
      expandedRegion = filteredRegions[0].name;
    }
  }

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
      
      const regionIdx = headers.indexOf('region');
      const pusIdx = headers.indexOf('pus');
      const longlatIdx = headers.indexOf('longlat');
      const statusIdx = headers.indexOf('statut');
      const addressIdx = headers.indexOf('adresse');
      const mapLinkIdx = headers.indexOf('map');

      agencies = rows.slice(1)
        .filter(row => row[statusIdx] === 'Live')
        .map(row => {
          const [lat, lng] = (row[longlatIdx] || '').split(',').map(Number);
          return {
            region: row[regionIdx],
            n: row[pusIdx],
            lat,
            lng,
            address: row[addressIdx],
            mapLink: row[mapLinkIdx]
          };
        })
        .filter(a => !isNaN(a.lat) && !isNaN(a.lng));

      updateMarkers();
    } catch (e) {
      console.error('Error fetching sheet data:', e);
    }
  }

  function updateMarkers() {
    if (markers && map) {
      markers.clearLayers();
      markerMap.clear();
      agencies.forEach(a => {
        const popupContent = `
          <div class="custom-popup">
            <div class="popup-header">
              <strong>${a.n}</strong>
            </div>
            <p class="popup-address">${a.address || 'Point Relais Jumia'}</p>
            <div class="popup-info">
              <div class="info-item">
                <span class="info-icon">🕒</span>
                <div>
                  <strong>Horaires</strong>
                  <span>Lun-Ven: 8h-18h | Sam: 9h-17h</span>
                </div>
              </div>
              <div class="info-item">
                <span class="info-icon">📞</span>
                <div>
                  <strong>Contact</strong>
                  <span>25 20 00 61 61</span>
                </div>
              </div>
            </div>
            <a href="${a.mapLink && a.mapLink.startsWith('http') ? a.mapLink : 'https://' + a.mapLink}" target="_blank" class="google-maps-btn">
              <span class="btn-icon">📍</span> Ouvrir sur Google Maps
            </a>
          </div>
        `;

        const m = L.marker([a.lat, a.lng], { icon: orangeIcon })
          .bindPopup(popupContent, { maxWidth: 300, minWidth: 260 });
        
        markers.addLayer(m);
        markerMap.set(a.n, m);
      });
      if (agencies.length > 0) {
        map.fitBounds(markers.getBounds().pad(0.1));
      }
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
          <div class="region-accordion">
            <button 
              class="region-header {expandedRegion === region.name ? 'expanded' : ''}" 
              on:click={() => toggleRegion(region.name, region.lat, region.lng, region.zoom)}
            >
              <span class="region-name">{region.name}</span>
              <span class="accordion-icon">{expandedRegion === region.name ? '−' : '+'}</span>
            </button>
            
            {#if expandedRegion === region.name}
              <div class="agency-list">
                {#each region.agencies as agency}
                  <button class="agency-item" on:click={() => handleAgencyClick(agency)}>
                    <div class="agency-info">
                      <strong class="agency-name">{agency.n}</strong>
                      <p class="agency-addr">{agency.address}</p>
                    </div>
                  </button>
                {/each}
              </div>
            {/if}
          </div>
        {/each}
      </div>
    </div>

    <div class="map-canvas-wrap">
      <div id="jumiaMap"></div>
    </div>
  </div>
</div>

<style>
  :global(.leaflet-popup-content-wrapper) {
    border-radius: 16px;
    padding: 0;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  }
  :global(.leaflet-popup-content) {
    margin: 0;
    width: auto !important;
  }
  :global(.custom-popup) {
    padding: 20px;
    font-family: 'Montserrat', sans-serif;
  }
  :global(.popup-header strong) {
    display: block;
    font-size: 16px;
    color: #1A1A1A;
    margin-bottom: 8px;
  }
  :global(.popup-address) {
    font-size: 13px;
    color: #666;
    line-height: 1.4;
    margin: 0 0 16px;
  }
  :global(.popup-info) {
    border-top: 1px solid #EEE;
    padding-top: 16px;
    margin-bottom: 20px;
  }
  :global(.info-item) {
    display: flex;
    gap: 12px;
    margin-bottom: 12px;
  }
  :global(.info-icon) {
    font-size: 18px;
    background: #F8F8F8;
    width: 36px;
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
  }
  :global(.info-item div) {
    display: flex;
    flex-direction: column;
  }
  :global(.info-item strong) {
    font-size: 12px;
    color: #1A1A1A;
  }
  :global(.info-item span) {
    font-size: 12px;
    color: #666;
  }
  :global(.google-maps-btn) {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    background: #111827;
    color: white !important;
    text-decoration: none;
    padding: 12px;
    border-radius: 12px;
    font-size: 14px;
    font-weight: 600;
    transition: background 0.2s;
  }
  :global(.google-maps-btn:hover) {
    background: #1F2937;
  }

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

  .region-list-container {
    flex: 1;
    overflow-y: auto;
    padding: 0 16px 20px;
  }

  .region-list-container::-webkit-scrollbar { width: 6px; }
  .region-list-container::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.1); border-radius: 10px; }

  .region-accordion {
    margin-bottom: 12px;
    background: white;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 4px 12px rgba(0,0,0,0.03);
  }

  .region-header {
    width: 100%;
    background: white;
    border: none;
    padding: 18px 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    cursor: pointer;
    text-align: left;
    transition: background 0.2s;
  }

  .region-header.expanded {
    border-bottom: 1px solid #F3F4F6;
  }

  .region-name {
    font-family: 'Montserrat', sans-serif;
    font-size: 14px;
    font-weight: 700;
    color: #111;
    line-height: 1.3;
    max-width: 85%;
  }

  .accordion-icon {
    font-size: 20px;
    color: #9CA3AF;
    font-weight: 300;
  }

  .agency-list {
    background: white;
    max-height: 400px;
    overflow-y: auto;
  }

  .agency-item {
    width: 100%;
    background: none;
    border: none;
    padding: 14px 20px;
    text-align: left;
    cursor: pointer;
    transition: background 0.15s;
    border-bottom: 1px solid #F9FAFB;
  }

  .agency-item:hover { background: #F3F4F6; }

  .agency-info { display: flex; flex-direction: column; gap: 4px; }
  .agency-name { font-family: 'Montserrat', sans-serif; font-size: 13.5px; font-weight: 700; color: #1F2937; }
  .agency-addr { font-size: 12px; color: #6B7280; line-height: 1.4; }

  /* Map Canvas */
  .map-canvas-wrap {
    flex: 1;
    background: #f8f8f8;
    position: relative;
  }

  #jumiaMap { width: 100%; height: 100%; }

  /* Custom Leaflet Pin */
  :global(.custom-pin) { background: none !important; border: none !important; }
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
  :global(.pin-inner) { width: 8px; height: 8px; background: #FFF; border-radius: 50%; }

  @media (max-width: 900px) {
    .map-container { flex-direction: column; height: auto; border-radius: 24px; }
    .map-sidebar { width: 100%; min-width: 100%; height: 450px; border-right: none; border-bottom: 1px solid rgba(0,0,0,0.05); }
    .map-canvas-wrap { height: 400px; }
  }
</style>
