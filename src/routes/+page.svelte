<!-- @ts-nocheck -->
<script>
  import { onMount } from 'svelte';
  import { marked } from 'marked';
  import episodesRaw from '$lib/episodes.json';

  /** Normalized and sorted episodes */
  let allEpisodes = [];

  /** Currently selected episode ID */
  let currentId = null;

  /** Rendered HTML of the current episode Markdown */
  let currentHtml = '';

  // Reactive current episode object
  $: currentEpisode = allEpisodes.find(e => e.id === currentId);

  // Compute Prev/Next IDs as reactive variables
  $: prevId = currentEpisode ? allEpisodes[allEpisodes.indexOf(currentEpisode) - 1]?.id ?? null : null;
  $: nextId = currentEpisode ? allEpisodes[allEpisodes.indexOf(currentEpisode) + 1]?.id ?? null : null;

  // Load episodes on mount: normalize IDs, sort, then load first
  onMount(async () => {
    allEpisodes = episodesRaw
      .map(e => ({ ...e, id: Number(e.id) }))
      .sort((a, b) => a.id - b.id);

    if (allEpisodes.length > 0) {
      await loadEpisode(allEpisodes[0].id);
    }
  });

  /** Load and render Markdown for a given episode ID */
  async function loadEpisode(id) {
    // Set currentId before fetching to update reactive deps
    currentId = id;

    const ep = allEpisodes.find(e => e.id === id);
    if (!ep) {
      currentHtml = `<p>Episode not found.</p>`;
      return;
    }

    try {
      const res = await fetch(ep.mdPath);
      if (!res.ok) throw new Error('Failed to fetch ' + ep.mdPath);
      const md = await res.text();
      currentHtml = await marked(md);
    } catch (err) {
      console.error(err);
      currentHtml = `<p>Error loading description.</p>`;
    }
  }

  /** Handle selection from list or navigation controls */
  function selectEpisode(id) {
    if (id != null && id !== currentId) {
      loadEpisode(id);
    }
  }
</script>

<div class="mfp-page">
  <!-- ASCII Visualizer on the left -->
  <aside class="visualizer">
    <!-- AsciiVisualizer goes here -->
  </aside>

  <!-- Main content -->
  <main class="content">
    {#if currentEpisode}
      <h1>{currentEpisode.title}</h1>
      <audio controls src={currentEpisode.audio}></audio>
      <div class="description">{@html currentHtml}</div>
    {:else}
      <p>Loading…</p>
    {/if}
  </main>

  <!-- Episode list on the right -->
  <aside class="episodes-list">
    <ul>
      {#each allEpisodes as ep}
        <li class:selected={ep.id === currentId}>
          <button type="button" on:click={() => selectEpisode(ep.id)}>
            {ep.id}. {ep.title}
          </button>
        </li>
      {/each}
    </ul>
  </aside>

  <!-- Navigation controls -->
  {#if currentEpisode}
    <div class="nav-controls">
      <button on:click={() => selectEpisode(prevId)} disabled={!prevId}>
        Prev
      </button>
      <button on:click={() => selectEpisode(nextId)} disabled={!nextId}>
        Next
      </button>
    </div>
  {/if}
</div>
