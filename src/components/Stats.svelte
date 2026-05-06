<script>
  export let stats = [];
</script>

<div class="stats-bar">
  <div class="stats-inner">
    {#each stats as stat}
      <div class="stat-item">
        <span class="stat-num">{stat.num}</span>
        <span class="stat-label">{stat.label}</span>
      </div>
    {/each}
  </div>
</div>

<style>
  .stats-bar { background: var(--orange); }
  .stats-inner { max-width: 1200px; margin: auto; padding: 0 32px; display: flex; }
  .stat-item { flex: 1; padding: 20px 0; text-align: center; border-right: 1px solid rgba(26, 18, 8, .15); }
  .stat-item:last-child { border-right: none; }
  .stat-num { font-family: 'Montserrat', sans-serif; font-size: 1.9rem; font-weight: 800; color: var(--dark); display: block; line-height: 1; }
  .stat-label { font-size: .78rem; font-weight: 500; color: rgba(26, 18, 8, .65); text-transform: uppercase; letter-spacing: .06em; margin-top: 4px; display: block; }

  @media (max-width: 600px) {
    .stats-inner { flex-wrap: wrap; }
    .stat-item { flex: 1 1 50%; border-right: none; border-bottom: 1px solid rgba(26, 18, 8, .1); }
  }
</style>
