<script>
  /**
   * `src` is the URL of the audio track.
   * `audioEl` is bound to the <audio> element in the parent.
   */
  export let src = '';
  export let audioEl;
</script>

<audio
  bind:this={audioEl}
  {src}
  controls
  style="width: 100%; margin: 1rem 0;"
></audio>
