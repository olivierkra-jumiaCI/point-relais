<script>
  import './index.css';
  import { stats, featuredCities, steps, agences, regions, cityFilterCoords, faqs } from './lib/data.js';
  
  import Hero from './components/Hero.svelte';
  import Stats from './components/Stats.svelte';
  import InfoBanner from './components/InfoBanner.svelte';
  import FeaturedCities from './components/FeaturedCities.svelte';
  import HowItWorks from './components/HowItWorks.svelte';
  import MapFinder from './components/MapFinder.svelte';
  import InfoCards from './components/InfoCards.svelte';
  import ZonesTable from './components/ZonesTable.svelte';
  import FAQ from './components/FAQ.svelte';
  import FooterCTA from './components/FooterCTA.svelte';
</script>

<svelte:head>
  <title>Agences & Points Relais — Jumia Côte d'Ivoire</title>
  <link rel="preconnect" href="https://fonts.googleapis.com"/>
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin/>
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700;800&family=Roboto:ital,wght@0,300;0,400;0,500;1,300&display=swap" rel="stylesheet"/>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.3/dist/leaflet.css"/>
  <script src="https://unpkg.com/leaflet@1.9.3/dist/leaflet.js"></script>
</svelte:head>

<div class="app-container">
  <Hero />
  <Stats {stats} />
  <InfoBanner />
  <FeaturedCities cities={featuredCities} />
  <HowItWorks {steps} />
  <MapFinder agencies={agences} {regions} {cityFilterCoords} />
  <InfoCards />
  <ZonesTable />
  <FAQ {faqs} />
  <FooterCTA />
</div>

<style>
  .app-container {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  @media (max-width: 600px) {
    .cities-grid, .steps-grid, .info-3col { grid-template-columns: 1fr; }
    .section, .info-grid-section, .zones-section, .faq-section, .footer-cta, .info-banner { padding: 0 16px; }
    .footer-cta-inner { flex-direction: column; }
  }
</style>
