<script>
  import { onMount, onDestroy } from 'svelte';
  import L from 'leaflet';
  import 'leaflet/dist/leaflet.css';

  export let agencies = [];
  export let regions = [];
  export let liveAgenciesList = [];

  let map;
  let markers;
  let searchQuery = '';
  let orangeIcon;
  let expandedRegion = null;
  let markerMap = new Map();
  let lastUpdateTimestamp = null;
  let isUpdating = false;
  let refreshInterval;

  function normalizeStr(str) {
    return (str || '').toString().trim().toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '');
  }

  $: allRegions = (() => {
    const regionMap = new Map();
    regions.forEach(r => {
      regionMap.set(normalizeStr(r.name), { ...r });
    });
    agencies.forEach(a => {
      if (a.region) {
        const norm = normalizeStr(a.region);
        if (!regionMap.has(norm)) {
          regionMap.set(norm, {
            name: a.region,
            lat: a.lat || 7.0,
            lng: a.lng || -5.5,
            zoom: 10
          });
        }
      }
    });
    return Array.from(regionMap.values());
  })();

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
    (a.n || '').toLowerCase().includes(searchQuery.toLowerCase()) ||
    (a.address || '').toLowerCase().includes(searchQuery.toLowerCase()) ||
    (a.region || '').toLowerCase().includes(searchQuery.toLowerCase())
  );

  $: filteredRegions = allRegions.map(region => {
    const agenciesInRegion = filteredAgencies.filter(a => normalizeStr(a.region) === normalizeStr(region.name));
    return {
      ...region,
      agencies: agenciesInRegion,
      visible: agenciesInRegion.length > 0
    };
  }).filter(r => r.visible).sort((a, b) => a.name.localeCompare(b.name, 'fr', { sensitivity: 'base' }));

  // Auto-expand if searching
  $: if (searchQuery.length > 0 && filteredRegions.length > 0) {
    if (!expandedRegion || !filteredRegions.find(r => normalizeStr(r.name) === normalizeStr(expandedRegion))) {
      expandedRegion = filteredRegions[0].name;
    }
  }

  function parseCoords(rawLatLong, mapLink) {
    let lat = NaN;
    let lng = NaN;

    if (rawLatLong) {
      const cleaned = rawLatLong.replace(/["']/g, '').trim();
      const parts = cleaned.split(',').map(s => s.trim());
      if (parts.length >= 2 && !isNaN(Number(parts[0])) && !isNaN(Number(parts[1]))) {
        lat = Number(parts[0]);
        lng = Number(parts[1]);
        if (lat !== 0 && lng !== 0) return { lat, lng };
      }

      const dmsMatch = cleaned.match(/(\d+)°\s*(\d+)[']?(\d+\.?\d*)["']?\s*([NS])[\s,]+(\d+)°\s*(\d+)[']?(\d+\.?\d*)["']?\s*([EW])/i);
      if (dmsMatch) {
        let dLat = Number(dmsMatch[1]) + Number(dmsMatch[2])/60 + Number(dmsMatch[3])/3600;
        if (dmsMatch[4].toUpperCase() === 'S') dLat = -dLat;
        let dLng = Number(dmsMatch[5]) + Number(dmsMatch[6])/60 + Number(dmsMatch[7])/3600;
        if (dmsMatch[8].toUpperCase() === 'W') dLng = -dLng;
        if (!isNaN(dLat) && !isNaN(dLng) && dLat !== 0 && dLng !== 0) {
          return { lat: Number(dLat.toFixed(6)), lng: Number(dLng.toFixed(6)) };
        }
      }
    }

    if (mapLink) {
      const match = mapLink.match(/(?:@|q=|ll=|\/place\/)([-+]?\d+\.\d+),([-+]?\d+\.\d+)/);
      if (match) {
        lat = Number(match[1]);
        lng = Number(match[2]);
        if (!isNaN(lat) && !isNaN(lng) && lat !== 0 && lng !== 0) return { lat, lng };
      }
    }

    return { lat: NaN, lng: NaN };
  }

  async function fetchSheetData(silent = false) {
    if (isUpdating) return;
    isUpdating = true;
    try {
      const response = await fetch(`https://docs.google.com/spreadsheets/d/1fxoAn0n25s8LFFvTWOT2tJIgYXLYhYc0RbZM66GqIxs/export?format=csv&gid=0&_t=${Date.now()}`, {
        cache: 'no-store'
      });
      const text = await response.text();
      
      const rows = text.split(/\r?\n/).map(row => {
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
      const headers = rows[0].map(h => h.trim().toLowerCase());
      
      const regionIdx = headers.findIndex(h => h === 'region');
      const pusIdx = headers.findIndex(h => h === 'pus' || h === 'nom' || h === 'repere');
      const longlatIdx = headers.findIndex(h => h === 'longlat' || h === 'latlong' || h.includes('long') || h.includes('lat'));
      const statusIdx = headers.findIndex(h => h === 'statut' || h === 'status' || h === 'état' || h === 'etat');
      const addressIdx = headers.findIndex(h => h === 'adresse' || h === 'address');
      const mapLinkIdx = headers.findIndex(h => h === 'map' || h === 'lien' || h === 'link');
      const grosColisIdx = headers.findIndex(h => h === 'groscolis' || h === 'gros colis');
      const nouveauIdx = headers.findIndex(h => h === 'nouveau' || h === 'new');

      const liveAgencies = rows.slice(1)
        .map(row => {
          const status = statusIdx >= 0 ? (row[statusIdx] || '').trim() : 'Live';
          const rawLatLong = (row[longlatIdx] || '').replace(/["']/g, '').trim();
          const mapLink = (row[mapLinkIdx] || '').replace(/["']/g, '').trim();
          const { lat, lng } = parseCoords(rawLatLong, mapLink);
          return {
            status,
            region: regionIdx >= 0 && (row[regionIdx] || '').trim() !== '' ? (row[regionIdx] || '').trim() : 'Autres',
            n: pusIdx >= 0 ? (row[pusIdx] || '').trim() : '',
            lat,
            lng,
            address: addressIdx >= 0 ? (row[addressIdx] || '').trim() : '',
            mapLink,
            grosColis: grosColisIdx >= 0 ? (row[grosColisIdx] || '').trim() : '',
            nouveau: nouveauIdx >= 0 ? (row[nouveauIdx] || '').trim() : 'Non'
          };
        })
        .filter(a => a.status.toLowerCase() === 'live' && a.n && !isNaN(a.lat) && !isNaN(a.lng));

      if (liveAgencies.length > 0) {
        agencies = liveAgencies;
        liveAgenciesList = liveAgencies;
        lastUpdateTimestamp = new Date();
        updateMarkers();
      }
    } catch (e) {
      console.error('Error fetching sheet data:', e);
    } finally {
      isUpdating = false;
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
            ${a.grosColis && a.grosColis.trim().toLowerCase() === 'non' ? `
              <div style="background: #fee2e2; color: #dc2626; padding: 6px 12px; border-radius: 8px; font-size: 12px; font-weight: 700; margin-bottom: 12px; display: flex; align-items: center; gap: 6px; border: 1px solid #fecaca;">
                <span style="font-size: 14px;">⚠️</span> Gros colis non accepté
              </div>
            ` : ''}
            <p class="popup-address">${a.address || 'Point Relais Jumia'}</p>
            <div class="popup-info">
              <div class="info-item">
                <span class="info-icon">🕒</span>
                <div>
                  <strong>Horaires</strong>
                  <span>Du Lundi au Samedi : 08h – 18h - Fermée les dimanches et jours fériés nationaux.</span>
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
      attributionControl: true 
    }).setView([7.0, -5.5], 6);

    L.control.zoom({ position: 'topright' }).addTo(map);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    orangeIcon = L.divIcon({
      className: 'custom-pin',
      html: '<div class="pin-wrap"><div class="pin-inner"></div></div>',
      iconSize: [28, 28], iconAnchor: [14, 28], popupAnchor: [0, -28]
    });

    markers = L.featureGroup().addTo(map);
    fetchSheetData(true);

    // Auto-refresh periodically while the page is open
    refreshInterval = setInterval(() => {
      if (typeof document !== 'undefined' && document.visibilityState === 'visible') {
        fetchSheetData(true);
      }
    }, 45000);

    const handleVisibilityChange = () => {
      if (typeof document !== 'undefined' && document.visibilityState === 'visible') {
        fetchSheetData(true);
      }
    };
    if (typeof document !== 'undefined') {
      document.addEventListener('visibilitychange', handleVisibilityChange);
    }
    if (typeof window !== 'undefined') {
      window.addEventListener('focus', () => fetchSheetData(true));
    }

    // Multiple invalidateSize calls to handle Android/mobile layout shifts
    [100, 300, 600, 1200].forEach(delay => {
      setTimeout(() => {
        if (map) map.invalidateSize();
      }, delay);
    });

    // Use ResizeObserver to re-trigger invalidateSize when container resizes
    const mapEl = document.getElementById('jumiaMap');
    if (mapEl && typeof ResizeObserver !== 'undefined') {
      const ro = new ResizeObserver(() => {
        if (map) map.invalidateSize();
      });
      ro.observe(mapEl);
    }
  });

  onDestroy(() => {
    if (refreshInterval) clearInterval(refreshInterval);
  });
</script>

<div class="map-finder-section" id="finder">
  <div class="map-finder-header">
    <h2>Nos Points Relais &amp; Zones d'Expédition</h2>
    <p>Plus de 200 Points Relais, déposer un colis est aussi simple que de marcher dans la rue.</p>
    <div class="live-sync-bar">
      <span class="live-badge"><span class="pulse-dot"></span> Données synchronisées en direct avec Google Sheets</span>
      {#if lastUpdateTimestamp}
        <span class="update-time">Mis à jour : {lastUpdateTimestamp.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}</span>
      {/if}
      <button class="refresh-btn {isUpdating ? 'spinning' : ''}" on:click={() => fetchSheetData(false)} title="Actualiser maintenant">
        🔄
      </button>
    </div>
    <div class="city-filters">
      <button class="city-filter {searchQuery === 'Abidjan' ? 'active' : ''}" on:click={() => searchQuery = 'Abidjan'}>Abidjan</button>
      <button class="city-filter {searchQuery === 'Yamoussoukro' ? 'active' : ''}" on:click={() => searchQuery = 'Yamoussoukro'}>Yamoussoukro</button>
      <button class="city-filter {searchQuery === 'Bouaké' ? 'active' : ''}" on:click={() => searchQuery = 'Bouaké'}>Bouaké</button>
      <button class="city-filter {searchQuery === 'San Pedro' ? 'active' : ''}" on:click={() => searchQuery = 'San Pedro'}>San Pedro</button>
      <button class="city-filter {searchQuery === '' ? 'active' : ''}" on:click={() => searchQuery = ''}>+100 Autres Villes</button>
    </div>
  </div>

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
  /* Leaflet Popup Overrides */
  :global(.premium-popup .leaflet-popup-content-wrapper) {
    border-radius: 24px;
    padding: 0;
    overflow: hidden;
    box-shadow: 0 20px 50px rgba(0,0,0,0.15);
    border: 1px solid rgba(0,0,0,0.05);
  }
  :global(.premium-popup .leaflet-popup-content) {
    margin: 0;
    width: auto !important;
  }
  :global(.premium-popup .leaflet-popup-tip-container) {
    display: none; /* Hide the tip for a cleaner card look */
  }
  :global(.premium-popup .leaflet-popup-close-button) {
    top: 15px !important;
    right: 15px !important;
    color: #999 !important;
    font-size: 20px !important;
    font-weight: 300 !important;
  }

  /* Custom Popup Content */
  :global(.custom-popup-content) {
    padding: 24px;
    font-family: 'Montserrat', sans-serif;
    color: #1A1A1A;
  }
  :global(.popup-title) {
    font-size: 19px;
    font-weight: 700;
    margin: 0 0 10px;
    padding-right: 20px;
  }
  :global(.popup-address) {
    font-size: 14px;
    color: #666;
    line-height: 1.5;
    margin: 0 0 20px;
    font-family: 'Roboto', sans-serif;
  }
  :global(.popup-details) {
    display: flex;
    flex-direction: column;
    gap: 16px;
    margin-bottom: 24px;
  }
  :global(.detail-row) {
    display: flex;
    align-items: flex-start;
    gap: 16px;
  }
  :global(.detail-icon-wrap) {
    width: 40px;
    height: 40px;
    background: #FFF9F0;
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #333;
    flex-shrink: 0;
  }
  :global(.detail-text) {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }
  :global(.detail-label) {
    font-size: 13px;
    font-weight: 700;
    color: #1A1A1A;
  }
  :global(.detail-value) {
    font-size: 13px;
    color: #666;
  }
  :global(.popup-cta) {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    background: #0F172A;
    color: #FFFFFF !important;
    text-decoration: none;
    padding: 15px;
    border-radius: 14px;
    font-size: 14px;
    font-weight: 700;
    transition: all 0.2s;
  }
  @media (hover: hover) {
    :global(.popup-cta:hover) {
      background: #1E293B;
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(15, 23, 42, 0.2);
    }
  }

  .map-finder-section {
    max-width: 1240px;
    margin: 40px auto;
    padding: 0 20px;
    width: 100%;
  }

  .map-finder-header {
    text-align: center;
    margin-bottom: 40px;
  }
  .map-finder-header h2 {
    font-family: 'Montserrat', sans-serif;
    font-size: 28px;
    font-weight: 800;
    color: #0F172A;
    margin: 0 0 16px;
  }
  .map-finder-header p {
    font-size: 16px;
    color: #64748B;
    margin: 0 0 32px;
  }
  .live-sync-bar {
    display: inline-flex;
    align-items: center;
    gap: 12px;
    background: #FFF9F0;
    border: 1px solid #FFD8A8;
    padding: 8px 18px;
    border-radius: 50px;
    margin-bottom: 24px;
    font-family: 'Montserrat', sans-serif;
    font-size: 13px;
    color: #333;
    box-shadow: 0 4px 12px rgba(255, 153, 0, 0.08);
  }
  .live-badge {
    display: flex;
    align-items: center;
    gap: 8px;
    font-weight: 700;
    color: #D97706;
  }
  .pulse-dot {
    width: 8px;
    height: 8px;
    background: #10B981;
    border-radius: 50%;
    box-shadow: 0 0 0 0 rgba(16, 185, 129, 0.7);
    animation: pulse 2s infinite;
  }
  @keyframes pulse {
    0% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(16, 185, 129, 0.7); }
    70% { transform: scale(1); box-shadow: 0 0 0 6px rgba(16, 185, 129, 0); }
    100% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(16, 185, 129, 0); }
  }
  .update-time {
    color: #64748B;
    font-size: 12px;
    border-left: 1px solid #E2E8F0;
    padding-left: 12px;
  }
  .refresh-btn {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 14px;
    padding: 4px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: transform 0.2s;
  }
  .refresh-btn:hover {
    transform: scale(1.2);
  }
  .refresh-btn.spinning {
    animation: spin 1s linear infinite;
  }
  @keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }
  .city-filters {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 12px;
  }
  .city-filter {
    background: #FFFFFF;
    border: 1px solid #E2E8F0;
    border-radius: 50px;
    padding: 10px 24px;
    font-family: 'Montserrat', sans-serif;
    font-size: 14px;
    font-weight: 500;
    color: #475569;
    cursor: pointer;
    transition: all 0.2s;
  }
  .city-filter:hover {
    border-color: #CBD5E1;
    background: #F8FAFC;
  }
  .city-filter.active {
    border-color: #FF9900;
    color: #FF9900;
    background: #FFF9F0;
  }

  .map-container {
    display: flex;
    background: var(--white);
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
    min-width: 0;
    max-width: 100%;
    display: flex;
    flex-direction: column;
    background: var(--white);
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
    .map-sidebar { width: 100%; min-width: 100%; height: 350px; border-right: none; border-bottom: 1px solid rgba(0,0,0,0.05); }
    .map-canvas-wrap { height: 350px; min-height: 350px; }
    #jumiaMap { width: 100%; height: 350px !important; min-height: 350px; }
  }
  @media (max-width: 600px) {
    .map-finder-section { margin: 24px auto; padding: 0 16px; }
    .map-canvas-wrap { height: 300px; min-height: 300px; }
    #jumiaMap { height: 300px !important; min-height: 300px; }
    .map-sidebar { height: 300px; }
  }
</style>
