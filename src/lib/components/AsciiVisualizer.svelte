<script>
  import { onMount, onDestroy } from 'svelte';

  /** The actual HTMLAudioElement bound in +page.svelte */
  export let audioEl;

  let canvasText = '';
  let analyser;
  let data;
  let frameId;

  onMount(() => {
    if (!audioEl) {
      console.warn('AsciiVisualizer: no audioEl passed');
      return;
    }

    const ctx = new AudioContext();
    const source = ctx.createMediaElementSource(audioEl);
    analyser = ctx.createAnalyser();
    analyser.fftSize = 64;

    // Tap the audio stream for analysis...
    source.connect(analyser);
    // ...but also send the audio to the speakers
    source.connect(ctx.destination);

    data = new Uint8Array(analyser.frequencyBinCount);
    drawFrame();
  });

  function drawFrame() {
    analyser.getByteFrequencyData(data);
    // Map first 32 bins into bar heights (0–8)
    const bars = Array.from(data.slice(0, 32)).map(v => Math.floor(v / 32));
    canvasText = bars.map(h => '▇'.repeat(h)).join('\n');
    frameId = requestAnimationFrame(drawFrame);
  }

  onDestroy(() => {
    if (frameId != null) cancelAnimationFrame(frameId);
  });
</script>

<pre class="ascii-visualizer" style="font-family: monospace; line-height: .75;">
{canvasText}
</pre>
