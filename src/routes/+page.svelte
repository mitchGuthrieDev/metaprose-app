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

  // Playback state
  let currentTime = 0;
  let duration = 0;
  let isPlaying = false;

  function formatTime(sec) {
    const m = Math.floor(sec / 60);
    const s = Math.floor(sec % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
  }

  $: currentEpisode = allEpisodes.find(e => e.id === currentId);
  $: prevId = currentEpisode ? allEpisodes[allEpisodes.indexOf(currentEpisode) - 1]?.id ?? null : null;
  $: nextId = currentEpisode ? allEpisodes[allEpisodes.indexOf(currentEpisode) + 1]?.id ?? null : null;

  onMount(async () => {
    allEpisodes = episodesRaw.map(e => ({ ...e, id: Number(e.id) })).sort((a, b) => a.id - b.id);
    if (allEpisodes.length) await loadEpisode(allEpisodes[0].id);

    audioEl.addEventListener('timeupdate', () => {
      currentTime = audioEl.currentTime;
    });
    audioEl.addEventListener('loadedmetadata', () => {
      duration = audioEl.duration;
    });
    audioEl.addEventListener('play', () => {
      isPlaying = true;
    });
    audioEl.addEventListener('pause', () => {
      isPlaying = false;
    });
    audioEl.addEventListener('ended', () => {
      isPlaying = false;
    });
  });

  async function loadEpisode(id) {
    currentId = id;
    const ep = allEpisodes.find(e => e.id === id);
    if (!ep) {
      currentHtml = `<p>Episode not found.</p>`;
      return;
    }
    const res = await fetch(ep.mdPath);
    const md = res.ok ? await res.text() : '';
    currentHtml = await marked(md);
    audioEl.currentTime = 0;
    currentTime = 0;
    duration = 0;
    isPlaying = false;
  }

  function togglePlay() {
    if (!audioEl) return;
    if (isPlaying) {
      audioEl.pause();
    } else {
      audioEl.play();
    }
  }

  function selectEpisode(id) {
    if (id != null && id !== currentId) loadEpisode(id);
  }
</script>

<div id="main">
  <div id="visualizer">
    {#if audioEl}
      <AsciiVisualizer {audioEl} />
    {/if}
  </div>

  <div id="content">
    {#if currentEpisode}
      <h1>Episode {currentEpisode.id}:<br />
          {currentEpisode.title}
      </h1>
      
      <!-- PLAYER UI -->
      <div id="player">
        <!-- Native audio hidden for analysis only -->
        <audio
          bind:this={audioEl}
          crossorigin="anonymous"
          src={currentEpisode.audio}
          style="display:none;"
        ></audio>

        <!-- Text‐based play/stop button -->
        <button
          type="button"
          class="bracket"
          on:click={togglePlay}
          class:active={isPlaying}
        >
          {#if isPlaying}[stop]{:else}[play]{/if}
        </button>

        <!-- Live time display -->
        <span class="time">
          {formatTime(currentTime)} / {formatTime(duration)}
        </span>
      </div>

      <div class="description">{@html currentHtml}</div>
    {:else}
      <p>Loading…</p>
    {/if}
  </div>

  <div id="episodes">
    <ul>
      {#each [...allEpisodes].reverse() as ep}
        <li>
          <button
            type="button"
            class="episode"
            class:selected={ep.id === currentId}
            on:click={() => {
              selectEpisode(ep.id);
              audioEl?.pause();
            }}
          >
            {ep.id}. {ep.title}
          </button>
        </li>
      {/each}
    </ul>
  </div>

  {#if currentEpisode}
    <div id="nav">
      <button on:click={() => { selectEpisode(prevId); audioEl?.pause(); }} disabled={!prevId}>
        Prev
      </button>
      <button on:click={() => { selectEpisode(nextId); audioEl?.pause(); }} disabled={!nextId}>
        Next
      </button>
    </div>
  {/if}
</div>
