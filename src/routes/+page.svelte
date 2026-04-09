<style>
  @import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,500;0,600;1,300;1,400;1,500&family=Jost:wght@200;300;400&display=swap');

  :global(*, *::before, *::after) { box-sizing: border-box; margin: 0; padding: 0; }
  :global(html, body) {
    height: 100%;
    background: #EDE8DF;
    color: #1A1410;
    font-family: 'Jost', sans-serif;
    font-weight: 300;
    /* Normal system cursor — no custom cursor */
    cursor: auto;
  }
  :global(html.scroll-locked) { overflow: hidden; }
  :global(html.scroll-unlocked) { overflow-y: auto; }

  /* ── HERO SECTION ───────────────────────────────────────────── */
  .hero {
    width: 100vw; height: 100vh;
    position: relative; overflow: hidden;
    background: #EDE8DF;
  }

  /* ── 3D VIEWER — absolute inset:0, sits behind everything ───── */
  .viewer-full {
    position: absolute; inset: 0; z-index: 1;
    /* Push canvas bottom up by the config bar height so the ring
       isn't hidden behind it. Config bar is ~130px. */
    bottom: 130px;
  }

  /* Subtle vignette — only the very outer rim, centre is clear */
  .atmo-overlay {
    position: absolute; z-index: 2; pointer-events: none;
    /* Only cover the sides and top — leave bottom open for config bar */
    top: 0; left: 0; right: 0;
    bottom: 130px;
    background:
      radial-gradient(ellipse 90% 90% at 50% 46%, transparent 65%, rgba(237,232,223,0.25) 100%);
  }

  /* ── TOP NAV ───────────────────────────────────────────────── */
  .nav {
    position: absolute; top: 0; left: 0; right: 0; z-index: 10;
    display: flex; align-items: center; justify-content: space-between;
    padding: 24px 40px;
    animation: fadeDown 1s ease 0.2s both;
  }
  .nav-logo {
    font-family: 'Cormorant Garamond', serif;
    font-size: 1.05rem; font-weight: 400; letter-spacing: 0.3em;
    text-transform: uppercase; color: rgba(30,20,10,0.75);
  }
  .nav-logo span { font-style: italic; color: rgba(140,105,50,0.85); }
  .nav-right { display: flex; align-items: center; gap: 32px; }
  .nav-links {
    display: flex; gap: 32px;
    font-size: 0.62rem; letter-spacing: 0.4em; text-transform: uppercase;
    color: rgba(30,20,10,0.4);
  }
  .nav-links span { transition: color 0.2s; }
  .nav-links span:hover { color: rgba(30,20,10,0.8); }

  .scroll-toggle {
    display: flex; align-items: center; gap: 8px;
    background: none; border: 1px solid rgba(140,105,50,0.25);
    padding: 6px 14px;
    font-family: 'Jost', sans-serif; font-size: 0.55rem;
    letter-spacing: 0.4em; text-transform: uppercase;
    color: rgba(140,105,50,0.7); transition: all 0.2s;
  }
  .scroll-toggle:hover { border-color: rgba(140,105,50,0.6); color: rgba(140,105,50,1); }
  .scroll-toggle svg { width: 11px; height: 11px; stroke: currentColor; fill: none; stroke-width: 1.5; transition: transform 0.3s; }
  .scroll-toggle.unlocked svg { transform: rotate(180deg); }

  /* ── RING TITLE ─────────────────────────────────────────────── */
  .ring-hero {
    position: absolute; top: 80px; left: 44px; z-index: 10;
    animation: fadeUp 1.2s ease 0.5s both;
  }
  .ring-eyebrow {
    font-size: 0.54rem; letter-spacing: 0.6em; text-transform: uppercase;
    color: rgba(140,105,50,0.7); margin-bottom: 6px;
    display: flex; align-items: center; gap: 10px;
  }
  .ring-eyebrow::before { content: ''; width: 22px; height: 1px; background: rgba(140,105,50,0.5); }
  .ring-title {
    font-family: 'Cormorant Garamond', serif;
    font-size: clamp(1.8rem, 3vw, 2.8rem);
    font-weight: 300; line-height: 1.0;
    color: #1A1410;
  }
  .ring-title em { font-style: italic; color: rgba(140,105,50,0.85); display: block; }

  /* ── CORNER ORNAMENTS ──────────────────────────────────────── */
  .orn { position: absolute; z-index: 5; pointer-events: none; width: 54px; height: 54px; }
  .orn-tl { top: 16px; left: 16px; }
  .orn-tr { top: 16px; right: 16px; transform: rotate(90deg); }
  .orn svg { width: 100%; height: 100%; }

  /* ── GRAIN ─────────────────────────────────────────────────── */
  .grain {
    position: absolute; inset: 0; z-index: 3; pointer-events: none; opacity: 0.022;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
    background-size: 128px 128px;
  }

  /* ── SCROLL INDICATOR ──────────────────────────────────────── */
  .scroll-hint {
    position: absolute; bottom: 148px; left: 50%; transform: translateX(-50%);
    z-index: 10; display: flex; flex-direction: column; align-items: center; gap: 6px;
    opacity: 0; pointer-events: none; transition: opacity 0.5s ease;
    font-size: 0.48rem; letter-spacing: 0.5em; text-transform: uppercase; color: rgba(140,105,50,0.5);
  }
  .scroll-hint.visible { opacity: 1; }
  .scroll-hint svg { width: 14px; height: 14px; stroke: rgba(140,105,50,0.5); fill: none; stroke-width: 1.5; animation: bounce 1.8s ease infinite; }
  @keyframes bounce { 0%,100% { transform: translateY(0); } 50% { transform: translateY(4px); } }

  /* ── BOTTOM CONFIG BAR — compact, sits at absolute bottom ──── */
  .config-bar {
    position: absolute; bottom: 0; left: 0; right: 0; z-index: 10;
    background: rgba(250,247,242,0.96);
    backdrop-filter: blur(20px) saturate(1.4);
    -webkit-backdrop-filter: blur(20px) saturate(1.4);
    border-top: 1px solid rgba(160,130,80,0.15);
    /* Compact padding — target ~130px total height */
    padding: 12px 44px 14px;
    animation: fadeUp 1s ease 0.8s both;
  }

  .config-tabs {
    display: flex; gap: 0; margin-bottom: 10px;
    border-bottom: 1px solid rgba(160,130,80,0.12);
  }
  .tab-btn {
    padding: 0 0 8px; margin-right: 24px;
    font-size: 0.58rem; letter-spacing: 0.42em; text-transform: uppercase;
    color: rgba(30,20,10,0.35); background: none; border: none;
    position: relative; transition: color 0.2s;
  }
  .tab-btn.active { color: rgba(140,105,50,0.9); }
  .tab-btn.active::after {
    content: ''; position: absolute; bottom: -1px; left: 0; right: 0;
    height: 1.5px; background: rgba(140,105,50,0.7);
  }

  .tab-panel { display: none; }
  .tab-panel.active { display: flex; align-items: center; gap: 32px; flex-wrap: wrap; }

  /* ── SWATCHES ──────────────────────────────────────────────── */
  .swatch-group { display: flex; flex-direction: column; gap: 8px; }
  .swatch-group-label {
    font-size: 0.53rem; letter-spacing: 0.42em; text-transform: uppercase;
    color: rgba(30,20,10,0.3);
  }
  .swatches { display: flex; gap: 9px; align-items: center; }
  .swatch {
    display: flex; flex-direction: column; align-items: center; gap: 5px;
    background: none; border: none; padding: 0;
  }
  .swatch-circle {
    width: 32px; height: 32px; border-radius: 50%;
    position: relative; transition: transform 0.2s ease;
    box-shadow: 0 2px 8px rgba(0,0,0,0.12), inset 0 1px 4px rgba(255,255,255,0.3);
  }
  .swatch:hover .swatch-circle { transform: scale(1.1); }
  .swatch-circle::after {
    content: ''; position: absolute; inset: -3px; border-radius: 50%;
    border: 1.5px solid transparent; transition: border-color 0.2s;
  }
  .swatch.active .swatch-circle::after { border-color: rgba(140,105,50,0.8); }
  .swatch-name {
    font-size: 0.47rem; letter-spacing: 0.28em; text-transform: uppercase;
    color: rgba(30,20,10,0.32); transition: color 0.2s;
  }
  .swatch.active .swatch-name { color: rgba(140,105,50,0.9); }

  .metal-gold     { background: radial-gradient(circle at 35% 35%, #EDD060, #B08820); }
  .metal-silver   { background: radial-gradient(circle at 35% 35%, #ECECEC, #A0A0A0); }
  .metal-rose     { background: radial-gradient(circle at 35% 35%, #D89898, #904858); }
  .metal-platinum { background: radial-gradient(circle at 35% 35%, #F0EEEC, #B0ADAA); }
  .diamond-white     { background: radial-gradient(circle at 35% 35%, #FFFFFF, #D0D0D8); border: 1px solid rgba(0,0,0,0.06); }
  .diamond-blue      { background: radial-gradient(circle at 35% 35%, #CCE8FF, #68A8D8); }
  .diamond-lightblue { background: radial-gradient(circle at 35% 35%, #E0F4FF, #90C8E0); }
  .diamond-pink      { background: radial-gradient(circle at 35% 35%, #FFD8E4, #E090A8); }
  .diamond-yellow    { background: radial-gradient(circle at 35% 35%, #FFFAC8, #E0C820); }

  /* ── SLIDER ────────────────────────────────────────────────── */
  .slider-group { display: flex; flex-direction: column; gap: 8px; min-width: 150px; }
  .slider-label-row { display: flex; justify-content: space-between; align-items: baseline; }
  .slider-label { font-size: 0.53rem; letter-spacing: 0.42em; text-transform: uppercase; color: rgba(30,20,10,0.3); }
  .slider-value { font-family: 'Cormorant Garamond', serif; font-size: 0.9rem; color: rgba(140,105,50,0.85); }
  input[type=range] {
    -webkit-appearance: none; width: 100%; height: 2px;
    background: rgba(140,105,50,0.2); border-radius: 1px; outline: none;
  }
  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none; width: 13px; height: 13px; border-radius: 50%;
    background: rgba(140,105,50,0.85);
    box-shadow: 0 1px 4px rgba(140,105,50,0.4); transition: transform 0.15s;
  }
  input[type=range]::-webkit-slider-thumb:hover { transform: scale(1.25); }

  /* ── TOGGLES ───────────────────────────────────────────────── */
  .toggle-group { display: flex; flex-direction: column; gap: 10px; }
  .toggle-row { display: flex; align-items: center; gap: 10px; }
  .toggle-label { font-size: 0.54rem; letter-spacing: 0.33em; text-transform: uppercase; color: rgba(30,20,10,0.45); }
  .toggle {
    width: 36px; height: 18px; border-radius: 9px;
    background: rgba(140,105,50,0.15); position: relative;
    transition: background 0.25s; flex-shrink: 0; border: none;
  }
  .toggle.on { background: rgba(140,105,50,0.75); }
  .toggle::after {
    content: ''; position: absolute; top: 2px; left: 2px;
    width: 14px; height: 14px; border-radius: 50%;
    background: #fff; transition: left 0.25s ease;
    box-shadow: 0 1px 4px rgba(0,0,0,0.18);
  }
  .toggle.on::after { left: 20px; }

  /* ── FINISH BUTTONS ────────────────────────────────────────── */
  .finish-btns { display: flex; gap: 7px; }
  .finish-btn {
    padding: 6px 14px; border-radius: 2px;
    border: 1px solid rgba(140,105,50,0.2); background: transparent;
    font-family: 'Jost', sans-serif; font-size: 0.54rem; letter-spacing: 0.32em;
    text-transform: uppercase; color: rgba(30,20,10,0.4); transition: all 0.2s;
  }
  .finish-btn:hover { border-color: rgba(140,105,50,0.5); color: rgba(140,105,50,0.8); }
  .finish-btn.active { background: rgba(140,105,50,0.08); border-color: rgba(140,105,50,0.7); color: rgba(140,105,50,0.9); }

  .mood-btns { display: flex; gap: 7px; flex-wrap: wrap; }
  .mood-btn {
    padding: 6px 14px; border-radius: 2px;
    border: 1px solid rgba(140,105,50,0.2); background: transparent;
    font-family: 'Jost', sans-serif; font-size: 0.54rem; letter-spacing: 0.28em;
    text-transform: uppercase; color: rgba(30,20,10,0.4); transition: all 0.2s;
    display: flex; align-items: center; gap: 7px;
  }
  .mood-btn:hover { border-color: rgba(140,105,50,0.5); color: rgba(140,105,50,0.8); }
  .mood-btn.active { background: rgba(140,105,50,0.08); border-color: rgba(140,105,50,0.7); color: rgba(140,105,50,0.9); }
  .mood-dot { width: 6px; height: 6px; border-radius: 50%; flex-shrink: 0; }

  /* ── RIGHT: PRICE + CTA ────────────────────────────────────── */
  .config-right {
    margin-left: auto; display: flex; flex-direction: column;
    align-items: flex-end; gap: 8px; flex-shrink: 0;
  }
  .price-row { display: flex; align-items: baseline; gap: 4px; }
  .price-cur { font-family: 'Cormorant Garamond', serif; font-size: 0.9rem; color: rgba(30,20,10,0.35); align-self: flex-start; margin-top: 2px; }
  .price-amt { font-family: 'Cormorant Garamond', serif; font-size: 2rem; font-weight: 300; color: #1A1410; letter-spacing: -0.01em; }
  .price-note { font-size: 0.5rem; letter-spacing: 0.22em; text-transform: uppercase; color: rgba(30,20,10,0.25); }
  .cta-row { display: flex; gap: 8px; }
  .btn-add {
    padding: 10px 20px;
    background: linear-gradient(135deg, #C9A848, #9F7818);
    border: none; color: #faf7f0;
    font-family: 'Jost', sans-serif; font-size: 0.56rem; font-weight: 400;
    letter-spacing: 0.42em; text-transform: uppercase;
    transition: all 0.25s ease; position: relative; overflow: hidden;
  }
  .btn-add::after { content: ''; position: absolute; inset: 0; background: linear-gradient(to bottom, rgba(255,255,255,0.1), transparent); }
  .btn-add:hover { transform: translateY(-1px); box-shadow: 0 6px 20px rgba(160,120,40,0.3); }
  .btn-save {
    padding: 9px 16px; background: transparent;
    border: 1px solid rgba(140,105,50,0.25); color: rgba(140,105,50,0.7);
    font-family: 'Jost', sans-serif; font-size: 0.56rem; letter-spacing: 0.42em;
    text-transform: uppercase; transition: all 0.25s;
  }
  .btn-save:hover { border-color: rgba(140,105,50,0.6); background: rgba(140,105,50,0.04); }

  /* ── LOADING SCREEN ────────────────────────────────────────── */
  .loader {
    position: fixed; inset: 0; z-index: 100;
    background: #EDE8DF; display: flex; flex-direction: column;
    align-items: center; justify-content: center; gap: 26px;
    transition: opacity 1.2s ease 0.3s, visibility 1.2s ease 0.3s;
  }
  .loader.gone { opacity: 0; visibility: hidden; pointer-events: none; }
  .loader-logo { font-family: 'Cormorant Garamond', serif; font-size: 1.35rem; font-weight: 300; letter-spacing: 0.5em; text-transform: uppercase; color: rgba(140,105,50,0.6); }
  .loader-bar { width: 150px; height: 1px; background: rgba(140,105,50,0.15); position: relative; overflow: hidden; }
  .loader-bar::after {
    content: ''; position: absolute; inset-block: 0; left: -100%; width: 100%;
    background: linear-gradient(to right, transparent, rgba(140,105,50,0.7), transparent);
    animation: scan 1.4s ease infinite;
  }
  @keyframes scan { to { left: 100%; } }
  .loader-label { font-size: 0.56rem; letter-spacing: 0.42em; text-transform: uppercase; color: rgba(140,105,50,0.4); }

  /* ── BELOW-FOLD ────────────────────────────────────────────── */
  .below-fold {
    background: #F5F1EA; padding: 100px 64px; min-height: 60vh;
    display: flex; gap: 72px; align-items: flex-start;
  }
  .bf-col { flex: 1; display: flex; flex-direction: column; gap: 20px; }
  .bf-label { font-size: 0.53rem; letter-spacing: 0.5em; text-transform: uppercase; color: rgba(140,105,50,0.6); }
  .bf-heading { font-family: 'Cormorant Garamond', serif; font-size: clamp(1.5rem, 2.2vw, 2.2rem); font-weight: 300; line-height: 1.2; color: #1A1410; }
  .bf-body { font-size: 0.76rem; line-height: 1.9; color: rgba(30,20,10,0.5); max-width: 360px; }
  .bf-divider { width: 1px; background: rgba(140,105,50,0.12); align-self: stretch; }

  /* ── ANIMATIONS ────────────────────────────────────────────── */
  @keyframes fadeDown { from { opacity:0; transform: translateY(-14px); } to { opacity:1; transform: none; } }
  @keyframes fadeUp   { from { opacity:0; transform: translateY(16px);  } to { opacity:1; transform: none; } }
</style>

<script>
  import { onMount } from 'svelte';
  import RingViewer from '$lib/components/RingViewer.svelte';

  let metalType    = 'gold';
  let diamondType  = 'white';
  let finish       = 'polished';
  let mood         = 'studio';
  let diamondSize  = 1.0;
  let sparkle      = true;
  let isLoaded     = false;

  let activeTab      = 'metal';
  let scrollUnlocked = false;

  const metals = [
    { id: 'gold',     label: '18k Gold',  cls: 'metal-gold' },
    { id: 'silver',   label: 'Silver',    cls: 'metal-silver' },
    { id: 'rose',     label: 'Rose Gold', cls: 'metal-rose' },
    { id: 'platinum', label: 'Platinum',  cls: 'metal-platinum' },
  ];
  const diamonds = [
    { id: 'white',     label: 'White',    cls: 'diamond-white' },
    { id: 'blue',      label: 'Sapphire', cls: 'diamond-blue' },
    { id: 'lightblue', label: 'Aqua',     cls: 'diamond-lightblue' },
    { id: 'pink',      label: 'Rosé',     cls: 'diamond-pink' },
    { id: 'yellow',    label: 'Canary',   cls: 'diamond-yellow' },
  ];
  const finishes = ['polished', 'brushed', 'matte'];
  const moods = [
    { id: 'studio',      label: 'Studio',      dot: '#D8D0C0' },
    { id: 'golden',      label: 'Golden Hour', dot: '#F0C040' },
    { id: 'twilight',    label: 'Twilight',    dot: '#8090C0' },
    { id: 'candlelight', label: 'Candlelight', dot: '#E06020' },
  ];

  const BASE_PRICES  = { gold: 4200, silver: 2800, rose: 3900, platinum: 5600 };
  const STONE_PRICE  = { white: 0, blue: 800, lightblue: 600, pink: 1100, yellow: 950 };
  const FINISH_PRICE = { polished: 0, brushed: 200, matte: 150 };

  $: sizeLabel  = diamondSize <= 1.005 ? 'Classic' : diamondSize <= 1.012 ? 'Classic+' : diamondSize <= 1.016 ? 'Grande' : 'Statement';
  $: caratLabel = (diamondSize).toFixed(2) + ' ct';

  $: baseP      = BASE_PRICES[metalType] ?? 0;
  $: stoneP     = STONE_PRICE[diamondType] ?? 0;
  $: finP       = FINISH_PRICE[finish] ?? 0;
  $: sizeP      = Math.round((baseP + stoneP) * ((diamondSize - 1) * 0.5));
  $: totalPrice = baseP + stoneP + finP + sizeP;

  $: if (typeof document !== 'undefined') {
    document.documentElement.classList.toggle('scroll-locked',  !scrollUnlocked);
    document.documentElement.classList.toggle('scroll-unlocked', scrollUnlocked);
  }

  onMount(() => {
    document.documentElement.classList.add('scroll-locked');
  });
</script>

<!-- Loading screen -->
<div class="loader" class:gone={isLoaded}>
  <div class="loader-logo">Hera-Nedit</div>
  <div class="loader-bar"></div>
  <div class="loader-label">Preparing your ring</div>
</div>

<div class="hero">

  <!-- 3D viewer — occupies hero minus config bar height at bottom -->
  <div class="viewer-full">
    <RingViewer
      bind:isLoaded
      {metalType} {diamondType} {finish} {mood} {diamondSize} {sparkle}
    />
  </div>

  <!-- Vignette — only covers the ring viewport area, not the config bar -->
  <div class="atmo-overlay"></div>
  <div class="grain"></div>

  <!-- Corner ornaments -->
  <div class="orn orn-tl">
    <svg viewBox="0 0 60 60" fill="none">
      <path d="M2 58V2H58" stroke="rgba(140,105,50,0.22)" stroke-width="1"/>
      <path d="M2 22V2H22" stroke="rgba(140,105,50,0.55)" stroke-width="1.5"/>
      <circle cx="2" cy="2" r="2" fill="rgba(140,105,50,0.55)"/>
    </svg>
  </div>
  <div class="orn orn-tr">
    <svg viewBox="0 0 60 60" fill="none">
      <path d="M2 58V2H58" stroke="rgba(140,105,50,0.22)" stroke-width="1"/>
      <path d="M2 22V2H22" stroke="rgba(140,105,50,0.55)" stroke-width="1.5"/>
      <circle cx="2" cy="2" r="2" fill="rgba(140,105,50,0.55)"/>
    </svg>
  </div>

  <!-- Nav -->
  <nav class="nav">
    <div class="nav-logo">Hera-Nedit <span>Jewellery</span></div>
    <div class="nav-right">
      <div class="nav-links">
        <span>Collections</span>
        <span>Atelier</span>
        <span>About</span>
        <span>Contact</span>
      </div>
      <button class="scroll-toggle" class:unlocked={scrollUnlocked} on:click={() => scrollUnlocked = !scrollUnlocked}>
        <svg viewBox="0 0 24 24"><polyline points="18 15 12 9 6 15"/></svg>
        {scrollUnlocked ? 'Lock View' : 'Explore'}
      </button>
    </div>
  </nav>

  <!-- Ring title -->
  <div class="ring-hero">
    <p class="ring-eyebrow">Bespoke Collection 2025</p>
    <h1 class="ring-title">
      Éternité
      <em>Double Band</em>
    </h1>
  </div>

  <!-- Scroll hint -->
  <div class="scroll-hint" class:visible={scrollUnlocked}>
    <span>Scroll</span>
    <svg viewBox="0 0 24 24"><polyline points="6 9 12 15 18 9"/></svg>
  </div>

  <!-- Config bar -->
  <div class="config-bar">
    <div class="config-tabs">
      {#each ['metal','stone','finish','size','scene'] as t}
        <button class="tab-btn" class:active={activeTab===t} on:click={() => activeTab=t}>
          {t === 'scene' ? 'Lighting & Scene' : t === 'size' ? 'Stone Size' : t.charAt(0).toUpperCase()+t.slice(1)}
        </button>
      {/each}
    </div>

    <div style="display:flex; align-items:flex-start; gap:0;">

      <div class="tab-panel" class:active={activeTab==='metal'}>
        <div class="swatch-group">
          <span class="swatch-group-label">Band Material</span>
          <div class="swatches">
            {#each metals as m}
              <button class="swatch" class:active={metalType===m.id} on:click={()=>metalType=m.id}>
                <div class="swatch-circle {m.cls}"></div>
                <span class="swatch-name">{m.label}</span>
              </button>
            {/each}
          </div>
        </div>
      </div>

      <div class="tab-panel" class:active={activeTab==='stone'}>
        <div class="swatch-group">
          <span class="swatch-group-label">Centre Stone</span>
          <div class="swatches">
            {#each diamonds as d}
              <button class="swatch" class:active={diamondType===d.id} on:click={()=>diamondType=d.id}>
                <div class="swatch-circle {d.cls}"></div>
                <span class="swatch-name">{d.label}</span>
              </button>
            {/each}
          </div>
        </div>
      </div>

      <div class="tab-panel" class:active={activeTab==='finish'}>
        <div class="swatch-group">
          <span class="swatch-group-label">Surface Finish</span>
          <div class="finish-btns">
            {#each finishes as f}
              <button class="finish-btn" class:active={finish===f} on:click={()=>finish=f}>
                {f.charAt(0).toUpperCase()+f.slice(1)}
              </button>
            {/each}
          </div>
        </div>
        <div style="width:1px;height:48px;background:rgba(140,105,50,0.1);margin:0 28px;"></div>
        <div class="toggle-group">
          <span class="swatch-group-label" style="margin-bottom:2px">Effects</span>
          <div class="toggle-row">
            <button class="toggle" class:on={sparkle} on:click={()=>sparkle=!sparkle}></button>
            <span class="toggle-label">Diamond Sparkle</span>
          </div>
        </div>
      </div>

      <div class="tab-panel" class:active={activeTab==='size'}>
        <div class="slider-group">
          <div class="slider-label-row">
            <span class="slider-label">Stone Size</span>
            <span class="slider-value">{sizeLabel} — {caratLabel}</span>
          </div>
          <input type="range" min="1.0" max="1.02" step="0.001" bind:value={diamondSize} />
        </div>
      </div>

      <div class="tab-panel" class:active={activeTab==='scene'}>
        <div class="swatch-group">
          <span class="swatch-group-label">Lighting Mood</span>
          <div class="mood-btns">
            {#each moods as m}
              <button class="mood-btn" class:active={mood===m.id} on:click={()=>mood=m.id}>
                <span class="mood-dot" style="background:{m.dot}"></span>
                {m.label}
              </button>
            {/each}
          </div>
        </div>
      </div>

      <div class="config-right">
        <div>
          <div class="price-row">
            <span class="price-cur">€</span>
            <span class="price-amt">{totalPrice.toLocaleString()}</span>
          </div>
          <div class="price-note">Excl. VAT · Free engraving</div>
        </div>
        <div class="cta-row">
          <button class="btn-save">♡ Save</button>
          <button class="btn-add">Add to Atelier →</button>
        </div>
      </div>

    </div>
  </div>

</div>

<!-- Below fold -->
<div class="below-fold">
  <div class="bf-col">
    <span class="bf-label">Craftsmanship</span>
    <h2 class="bf-heading">Each ring is<br/>made by hand</h2>
    <p class="bf-body">Our artisans spend up to forty hours on a single piece. Every setting is individually checked under magnification before it leaves the atelier.</p>
  </div>
  <div class="bf-divider"></div>
  <div class="bf-col">
    <span class="bf-label">Materials</span>
    <h2 class="bf-heading">Ethically sourced<br/>gemstones</h2>
    <p class="bf-body">We work exclusively with certified suppliers and traceable supply chains — so every stone you choose carries a story worth telling.</p>
  </div>
  <div class="bf-divider"></div>
  <div class="bf-col">
    <span class="bf-label">Delivery</span>
    <h2 class="bf-heading">Delivered in<br/>8 – 12 weeks</h2>
    <p class="bf-body">Your bespoke ring is made to order. We ship in a handcrafted wooden case with a certificate of authenticity and lifetime resizing included.</p>
  </div>
</div>