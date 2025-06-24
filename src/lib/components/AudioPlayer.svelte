<script>
  export let src = '';
  export let audioEl;
</script>

<audio
  bind:this={audioEl}
  controls
  crossorigin="anonymous"
  {src}
  style="width: 100%; margin: 1rem 0;"
></audio>
