<script>
  import { onMount, onDestroy } from 'svelte';

  export let audioEl;

  let analyser;
  let data;
  let frameId;
  let ctx;

  let columns = 50;
  const rows = 8;
  const charMap = [' ', '.', '·', 'o', '*', '°', '^', '#'];
  const peakChars = ['#', '@', '%', '$', '&', '*'];
  const colorMap = ['grey', 'yellow', 'orange', 'fuschia', 'purple', 'blue', 'green', 'white'];

  let displayRows = Array(rows).fill('');

  function applyFletcherMunson(index, totalBins) {
    const x = index / totalBins;
    return Math.exp(-4 * (x - 0.4) ** 2) + 0.5;
  }

  function drawFrame() {
    analyser.getByteFrequencyData(data);
    const binSize = Math.floor(data.length / columns);

    let bins = [];
    for (let i = 0; i < columns; i++) {
      const start = i * binSize;
      const end = start + binSize;
      let sum = 0;
      for (let j = start; j < end; j++) {
        const weight = applyFletcherMunson(j, data.length);
        sum += data[j] * weight;
      }
      bins.push((sum / binSize) / 255);
    }

    displayRows = [];
    for (let row = rows; row >= 1; row--) {
      let line = '';
      bins.forEach(val => {
        const level = Math.floor(val * rows);
        if (level >= row) {
          let char = charMap[Math.min(row, charMap.length - 1)];
          if (level >= rows - 1 && Math.random() < 0.3) {
            char = peakChars[Math.floor(Math.random() * peakChars.length)];
          }
          const colorClass = colorMap[Math.min(row, colorMap.length - 1)];
          line += `<span class="${colorClass}">${char}</span>`;
        } else {
          line += '&nbsp;';
        }
      });
      displayRows.push(line);
    }

    frameId = requestAnimationFrame(drawFrame);
  }

  onMount(() => {
    if (!audioEl) {
      console.warn('AsciiVisualizer: no audioEl passed');
      return;
    }

    columns = Math.floor(window.innerWidth / 15);

    ctx = new AudioContext();
    const source = ctx.createMediaElementSource(audioEl);
    analyser = ctx.createAnalyser();
    analyser.fftSize = 1024;
    analyser.smoothingTimeConstant = 0.85;

    source.connect(analyser);
    source.connect(ctx.destination);

    data = new Uint8Array(analyser.frequencyBinCount);

    audioEl.addEventListener('play', () => {
      if (ctx.state === 'suspended') ctx.resume();
    }, { once: true });

    drawFrame();

    window.addEventListener('resize', handleResize);
  });

  function handleResize() {
    columns = Math.floor(window.innerWidth / 15);
  }

  onDestroy(() => {
    cancelAnimationFrame(frameId);
    audioEl && audioEl.removeEventListener('play', ctx.resume);
    window.removeEventListener('resize', handleResize);
  });
</script>

<pre class="ascii-visualizer" style="font-family: monospace; line-height: 1;">
  {@html displayRows.join('<br>')}
</pre>

<style>
  .ascii-visualizer span {
    display: inline-block;
    width: 1ch;
  }
</style>
