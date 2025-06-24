<!-- @ts-nocheck -->
<script>
  import { onMount } from 'svelte';
  import { marked } from 'marked';
  import AsciiVisualizer from '$lib/components/AsciiVisualizer.svelte';
  import episodesRaw from '$lib/episodes.json';

  // Theme invert state
  let isInverted = false;
  function toggleInvert() {
    isInverted = !isInverted;
    document.body.classList.toggle('invert', isInverted);
  }

  // View & data state
  let viewType = 'episode';
  let allEpisodes = [];
  let currentId = null;
  let currentHtml = '';

  // Audio state
  let audioEl;
  let currentTime = 0;
  let duration = 0;
  let isPlaying = false;
  let fileSize = '';

  // Page markdown paths
  const pages = { about: '/pages/about.md', credits: '/pages/credits.md' };

  // Format seconds as M:SS
  function formatTime(sec) {
    const m = Math.floor(sec / 60);
    const s = Math.floor(sec % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
  }

  // Reactive: currentEpisode, prev/next IDs
  $: currentEpisode = viewType === 'episode' && currentId != null
    ? allEpisodes.find(e => e.id === currentId)
    : null;

  $: prevId = viewType === 'episode' && currentEpisode
    ? allEpisodes[allEpisodes.indexOf(currentEpisode) - 1]?.id ?? null
    : null;

  $: nextId = viewType === 'episode' && currentEpisode
    ? allEpisodes[allEpisodes.indexOf(currentEpisode) + 1]?.id ?? null
    : null;

  // Initialize episodes and default page
  onMount(async () => {
    allEpisodes = episodesRaw
      .map(e => ({ ...e, id: Number(e.id) }))
      .sort((a, b) => a.id - b.id);
    await loadPage('episode');
  });

  // Load either an episode or an info page
  async function loadPage(page) {
    viewType = page;
    if (page === 'episode') {
      if (currentId == null && allEpisodes.length) {
        currentId = allEpisodes[0].id;
      }
      await loadEpisode(currentId);
    } else {
      // Load Markdown page
      currentHtml = '<p>Loading…</p>';
      const res = await fetch(pages[page]);
      const md = res.ok ? await res.text() : `# ${page} not found`;
      currentHtml = await marked(md);
      audioEl?.pause();
    }
  }

  // Load a specific episode
  async function loadEpisode(id) {
    viewType = 'episode';
    currentId = id;
    const ep = allEpisodes.find(e => e.id === id);
    if (!ep) {
      currentHtml = `<p>Episode not found.</p>`;
      return;
    }
    // Fetch episode markdown
    const res = await fetch(ep.mdPath);
    const md = res.ok ? await res.text() : '';
    currentHtml = await marked(md);

    // Reset audio
    audioEl.currentTime = 0;
    currentTime = 0;
    duration = 0;
    isPlaying = false;

    // Fetch file size via HEAD
    try {
      const head = await fetch(ep.audio, { method: 'HEAD' });
      const len = head.headers.get('content-length');
      const url = audioEl?.src || ep.audio;
      console.log('Fetched content-length for', url, ':', len);
      fileSize = len ? `${(Number(len) / 1024).toFixed(1)} KB` : '';
    } catch {
      fileSize = '';
    }
  }

  // Toggle audio play/pause
  function togglePlay() {
    if (!audioEl) return;
    isPlaying ? audioEl.pause() : audioEl.play();
  }

  // Select an episode
  function selectEpisode(id) {
    if (viewType !== 'episode' || id !== currentId) {
      loadEpisode(id);
    }
  }
</script>

<div id="main">
  <!-- Top bracket nav -->
  <div id="topnav">
    <button class="btn btn-nav-about" on:click={() => loadPage('about')}>[about]</button>
    <button class="btn btn-nav-credits" on:click={() => loadPage('credits')}>[credits]</button>
    <button class="btn btn-nav-rss" on:click={() => window.open('/rss.xml', '_blank')}>[rss]</button>
    <button class="btn btn-nav-source" on:click={() => window.open('https://github.com/your/repo', '_blank')}>[source]</button>
    <button class="btn btn-invert" on:click={toggleInvert}>[invert]</button>
  </div>

  {#if viewType === 'episode'}
    <div id="visualizer">
      {#if audioEl}
        <AsciiVisualizer {audioEl} />
      {/if}
    </div>
  {/if}

  <div id="content">
    <h1>
      {#if viewType === 'episode' && currentEpisode}
        Episode {currentEpisode.id}:<br />
        {currentEpisode.title}
      {:else}
        {viewType.charAt(0).toUpperCase() + viewType.slice(1)}
      {/if}
    </h1>

    {#if viewType === 'episode' && currentEpisode}
      <div id="player">
        <audio
          bind:this={audioEl}
          crossorigin="anonymous"
          src={currentEpisode.audio}
          style="display:none;"
          on:timeupdate={() => (currentTime = audioEl.currentTime)}
          on:loadedmetadata={() => (duration = audioEl.duration)}
          on:play={() => (isPlaying = true)}
          on:pause={() => (isPlaying = false)}
          on:ended={() => (isPlaying = false)}
        ></audio>
        <button type="button" class="btn btn-play" on:click={togglePlay}>{isPlaying ? '[stop]' : '[play]'}</button>
        <span class="time">{formatTime(currentTime)} / {formatTime(duration)}</span>
        <div class="source-download">
          <button class="btn btn-source" on:click={() => (window.location.href = currentEpisode.audio)}>[source]</button>
          {#if fileSize}
            <span class="file-size">{fileSize}</span>
          {/if}
        </div>
      </div>
    {/if}

    <div class="description">{@html currentHtml}</div>
  </div>

  <div id="episodes">
    <ul>
      {#each [...allEpisodes].reverse() as ep}
        <li>
          <button
            type="button"
            class="btn btn-episode"
            class:selected={viewType === 'episode' && ep.id === currentId}
            on:click={() => selectEpisode(ep.id)}
          >
            {ep.id}. {ep.title}
          </button>
        </li>
      {/each}
    </ul>
  </div>

  {#if viewType === 'episode'}
    <div id="nav">
      <button class="btn btn-nav-about" on:click={() => selectEpisode(prevId)} disabled={!prevId}>Prev</button>
      <button class="btn btn-nav-about" on:click={() => selectEpisode(nextId)} disabled={!nextId}>Next</button>
    </div>
  {/if}
</div>
