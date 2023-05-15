<script>
  import { onMount, onDestroy } from "svelte";
  // import {PitchDetector} from 'pitchy'
  // const detector = PitchDetector.forFloat32Array(1024)

  let n = 0;
  let ded = false;
  onMount(() => {
    const u = () => {
      if (ded) return;
      const cycle = ((1000 * 60) / 119) * 4;
      n = Math.round(Date.now() % cycle) / cycle;
      requestAnimationFrame(u);
    };
    u();
  });
  onDestroy(() => {
    ded = true;
  });

  $: {
    document.body.dataset.beat = Math.floor(1 + n * 4);
  }
  let maxBucket = 0;
  let sumVel = 0;
  const onMic = (e) => {
    const o = e.detail;
    let sum = 0,
      count = 0,
      sum2 = 0;
    for (let i = 0; i < o.length; i++) {
      sum += o[i] * i;
      count += o[i];
      sum2 += o[i];
    }
    sumVel = sum2;
    maxBucket = sum / count;
  };
</script>

<div
  style="position:fixed;top:0;left:0;color:#ccc;font:50px consolas,monospace;opacity:0.2"
>
  debug_info:
  {Math.floor(1 + n * 4)} ({"#".repeat(Math.round(sumVel / 1000))})
</div>

<svelte:body on:mic={onMic} />

<div
  style="position:absolute;left:{maxBucket}vw;height:300px;width:10px; background:
  linear-gradient(to bottom,#f002,#f000);top:0;color"
/>
