<!-- @ts-nocheck -->
<script>
  import { onMount } from 'svelte';
  import { marked } from 'marked';
  import AsciiVisualizer from '$lib/components/AsciiVisualizer.svelte';
  import episodesRaw from '$lib/episodes.json';

  let allEpisodes = [];
  let currentId = null;
  let currentHtml = '';
  let audioEl;

  $: currentEpisode = allEpisodes.find(e => e.id === currentId);
  $: prevId = currentEpisode ? allEpisodes[allEpisodes.indexOf(currentEpisode) - 1]?.id ?? null : null;
  $: nextId = currentEpisode ? allEpisodes[allEpisodes.indexOf(currentEpisode) + 1]?.id ?? null : null;

  onMount(async () => {
    console.log('Log Output Success');
    allEpisodes = episodesRaw
      .map(e => ({ ...e, id: Number(e.id) }))
      .sort((a, b) => a.id - b.id);
    if (allEpisodes.length) {
      await loadEpisode(allEpisodes[0].id);
    }
  });

  async function loadEpisode(id) {
    currentId = id;
    const ep = allEpisodes.find(e => e.id === id);
    if (!ep) {
      currentHtml = `<p>Episode not found.</p>`;
      return;
    }
    try {
      const res = await fetch(ep.mdPath);
      const md = res.ok ? await res.text() : '';
      currentHtml = await marked(md);
      audioEl.currentTime = 0;
    } catch {
      currentHtml = `<p>Error loading description.</p>`;
    }
  }

  function selectEpisode(id) {
    if (id != null && id !== currentId) loadEpisode(id);
  }
</script>

<div id="main">
  <!-- Main content & audio player first for visualizer dependency -->
  <div id="content">
    {#if currentEpisode}
      <h1>{currentEpisode.title}</h1>
      <div id="player">
        <audio
          bind:this={audioEl}
            controls
            src={currentEpisode.audio}
            on:loadedmetadata={() => {
            // Sanity check right when the media metadata is loaded:
            console.log('Bound audioEl:', audioEl);
            console.log('audioEl.src =', audioEl?.src);
          }}
        ></audio>
      </div>
      <div class="description">{@html currentHtml}</div>
    {:else}
      <p>Loading…</p>
    {/if}
  </div>

  <!-- Visualizer section -->
  <div id="visualizer">
    {#if audioEl}
      <AsciiVisualizer audioEl={audioEl} />
    {/if}
  </div>

  <!-- Episodes list -->
  <div id="episodes">
    <ul>
      {#each allEpisodes as ep}
        <li class:selected={ep.id === currentId}>
          <button
            type="button"
            on:click={() => selectEpisode(ep.id)}
          >
            {ep.id}. {ep.title}
          </button>
        </li>
      {/each}
    </ul>
  </div>

  <!-- Navigation controls -->
  {#if currentEpisode}
    <div id="nav">
      <button on:click={() => selectEpisode(prevId)} disabled={!prevId}>
        Prev
      </button>
      <button on:click={() => selectEpisode(nextId)} disabled={!nextId}>
        Next
      </button>
    </div>
  {/if}
</div>
