<script>
  import { onMount } from 'svelte';
  import { marked } from 'marked';
  import AsciiVisualizer from '$lib/components/AsciiVisualizer.svelte';
  import episodesRaw from '$lib/episodes.json';

  // Theme state
  let isInverted = false;
  let isGrayscale = false;

  function toggleInvert() {
    isInverted = !isInverted;
    document.body.classList.toggle('invert', isInverted);
  }

  function toggleGrayscale() {
    isGrayscale = !isGrayscale;
    document.body.classList.toggle('grayscale', isGrayscale);
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

  const pages = { about: '/pages/about.md', credits: '/pages/credits.md' };

  function formatTime(sec) {
    const m = Math.floor(sec / 60);
    const s = Math.floor(sec % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
  }

  $: currentEpisode = viewType === 'episode' && currentId != null
    ? allEpisodes.find(e => e.id === currentId)
    : null;

  $: prevId = viewType === 'episode' && currentEpisode
    ? allEpisodes[allEpisodes.indexOf(currentEpisode) - 1]?.id ?? null
    : null;

  $: nextId = viewType === 'episode' && currentEpisode
    ? allEpisodes[allEpisodes.indexOf(currentEpisode) + 1]?.id ?? null
    : null;

  onMount(async () => {
    allEpisodes = episodesRaw.map(e => ({ ...e, id: Number(e.id) })).sort((a, b) => a.id - b.id);
    await loadPage('episode');
  });

  async function loadPage(page) {
    viewType = page;
    if (page === 'episode') {
      if (currentId == null && allEpisodes.length) {
        currentId = allEpisodes[0].id;
      }
      await loadEpisode(currentId);
    } else {
      currentHtml = '<p>Loading…</p>';
      const res = await fetch(pages[page]);
      const md = res.ok ? await res.text() : `# ${page} not found`;
      currentHtml = await marked(md);
      audioEl?.pause();
    }
  }

  async function loadEpisode(id) {
    viewType = 'episode';
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

    try {
      const head = await fetch(ep.audio, { method: 'HEAD' });
      const len = head.headers.get('content-length');
      fileSize = len ? `${(Number(len) / (1024 * 1024)).toFixed(2)} MB` : '';
    } catch {
      fileSize = '';
    }
  }

  function togglePlay() {
    if (!audioEl) return;
    isPlaying ? audioEl.pause() : audioEl.play();
  }

  function selectEpisode(id) {
    if (viewType !== 'episode' || id !== currentId) {
      loadEpisode(id);
    }
  }

  function seek(seconds) {
  if (!audioEl) return;
  let target = audioEl.currentTime + seconds;
  if (target < 0) target = 0;
  if (target > audioEl.duration) target = audioEl.duration;
  audioEl.currentTime = target;
}

</script>

<main class="full h-min-screen">
  <div class="px">
    <div class="flex row wrap">

      <!-- LEFT SIDEBAR -->
      <section class="full md:half lg:quarter lg:screen-v-scroll flex col">
        <header class="full noselect pt">
          <div class="pad">
            <div class="green">
              <em>scalineSelects</em>();
            </div>
            <div class="grey">
              A series of mixes from Scaline Audio's favorite artists.
            </div>
          </div>
        </header>

        <nav class="full noselect">
          {#if viewType === 'episode' && audioEl}
          <div id="visualizer" class="pad">
            <AsciiVisualizer {audioEl} />
          </div>
          {/if}

          <div id="controls-1">
            <button class="btn btn-audio" on:click={() => selectEpisode(prevId)} disabled={!prevId}>[prev]</button>
            <button class="btn btn-audio" on:click={() => seek(-30)} disabled={!audioEl}>[-30]</button>
            <button type="button" class="btn btn-audio" on:click={togglePlay}>{isPlaying ? '[stop]' : '[play]'}</button>
            <button class="btn btn-audio" on:click={() => seek(30)} disabled={!audioEl}>[+30]</button>
            <button class="btn btn-audio" on:click={() => selectEpisode(nextId)} disabled={!nextId}>[next]</button>
          </div>


          <div id="controls-2">
            <button class="btn btn-volume" on:click={() => loadPage('episode')}>[v-]</button>
            <button class="btn btn-volume" on:click={() => loadPage('about')}>[v-]</button>
            <button class="btn btn-volume" on:click={() => loadPage('credits')}>[random]</button>
          </div>
        </nav>

        <nav class="full noselect grey">
          <div id="nav">
            <button class="btn btn-audio" on:click={() => loadPage('about')}>[about]</button>
            <button class="btn btn-nav-credits" on:click={() => loadPage('credits')}>[credits]</button>
            <button class="btn btn-nav-rss" on:click={() => window.open('/rss.xml', '_blank')}>[rss]</button>
            <button class="btn btn-nav-source" on:click={() => window.open('https://www.paypal.com/donate/?hosted_button_id=283D6N3NJJ7T2&sdkMeta=eyJ1cmwiOiJodHRwczovL3d3dy5wYXlwYWxvYmplY3RzLmNvbS9kb25hdGUvc2RrL2RvbmF0ZS1zZGsuanMiLCJhdHRycyI6eyJkYXRhLXVpZCI6InVpZF9wb2t1aW9tbmJnc293cGhpc2F1Z2VianVpb21iamsifX0&targetMeta=eyJ6b2lkVmVyc2lvbiI6IjlfMF81OCIsInRhcmdldCI6IkRPTkFURSIsInNka1ZlcnNpb24iOiIwLjkuMCJ9', '_blank')}>[donate]</button>
            <button class="btn btn-invert" on:click={toggleInvert}>[invert]</button>
            <button class="btn btn-invert" on:click={toggleGrayscale}>[grayscale]</button>
          </div>
        </nav>
      </section>

      <!-- MAIN CONTENT -->
      <section id="content" class="full md:half lg:screen-v-scroll flex row wrap relative">
        <div class="full md:py">
          <header class="full noselect pt">
            <div class="pad pl-1">
              <div class="large title white">
                {#if viewType === 'episode' && currentEpisode}
                  Episode {currentEpisode.id}:<br />
                  {currentEpisode.title}
                {:else}
                  {viewType.charAt(0).toUpperCase() + viewType.slice(1)}
                {/if}
              </div>
            </div>
          </header>

          <article class="pl-1">
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
                <button class="btn btn-audio" on:click={togglePlay}>{isPlaying ? '[stop]' : '[play]'}</button>
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
          </article>
        </div>
      </section>

      <!-- EPISODES SIDEBAR -->
      <section class="full md:hidden lg:flex lg:quarter lg:screen-v-scroll flex col">
        <header class="full noselect pt">
          <div class="pl-1">&nbsp;</div>
        </header>

        <aside class="full noselect py">
          <div class="flex col">
            {#each [...allEpisodes].reverse() as ep}
              <div class="mb-1">
                <button
                  class="btn btn-episode text-sm truncate"
                  class:selected={viewType === 'episode' && ep.id === currentId}
                  on:click={() => selectEpisode(ep.id)}
                >
                  {ep.id}: {ep.title}
                </button>
              </div>
            {/each}
          </div>
        </aside>
      </section>

    </div>
  </div>
</main>
