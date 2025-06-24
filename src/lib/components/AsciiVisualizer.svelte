<script>
  import { onMount, onDestroy } from 'svelte';

  /** The HTMLAudioElement bound in +page.svelte */
  export let audioEl;

  let canvasText = '';
  let analyser;
  let data;
  let frameId;
  let ctx;

  function drawFrame() {
    analyser.getByteFrequencyData(data);
    const bars = Array.from(data.slice(0, 32)).map(v => Math.floor(v / 32));
    canvasText = bars.map(h => '▇'.repeat(h)).join('\n');
    frameId = requestAnimationFrame(drawFrame);
  }

  onMount(() => {
    if (!audioEl) {
      console.warn('AsciiVisualizer: no audioEl passed');
      return;
    }

    ctx = new AudioContext();
    const source = ctx.createMediaElementSource(audioEl);
    analyser = ctx.createAnalyser();
    analyser.fftSize = 64;

    // Tap the stream for analysis:
    source.connect(analyser);
    // AND also send audio to speakers:
    source.connect(ctx.destination);

    data = new Uint8Array(analyser.frequencyBinCount);

    // When the user hits play, resume the context so sound flows
    audioEl.addEventListener('play', () => {
      if (ctx.state === 'suspended') {
        ctx.resume();
      }
    }, { once: true });

    drawFrame();
  });

  onDestroy(() => {
    cancelAnimationFrame(frameId);
    audioEl && audioEl.removeEventListener('play', ctx.resume);
  });
</script>

<pre class="ascii-visualizer" style="font-family: monospace; line-height: .75;">
{canvasText}
</pre>
