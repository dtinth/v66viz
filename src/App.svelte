<script>
  import { onMount, onDestroy } from "svelte";
  import Beat from "./Beat.svelte";
  let data = [];
  let index = 0;
  let latest = [];
  const votes = {};

  // returns an index
  const weightedRandom = (weights) => {
    const total = weights.reduce((a, b) => a + b);
    const r = Math.random() * total;
    let i = 0;
    let acc = 0;
    while (acc < r) {
      acc += weights[i];
      i++;
    }
    return i - 1;
  };

  let ded = false;
  onMount(() => {
    const refre = () => {
      if (ded) return;
      fetch("/data.json")
        .then((r) => {
          return r.json();
        })
        .then((r) => {
          latest = r;
        });
    };
    refre();
    setInterval(refre, 30000);
    setInterval(() => {
      if (ded) return;
      const weights = latest.map((x) => x.count_total_seats);
      const x = latest[weightedRandom(weights)];
      if (!x) return;
      index++;
      const target = x.count_partylist_good_votes;
      const current = votes[x.id] || target * 0.75;
      const diff = target - current;
      const next = current + diff * 0.01;
      votes[x.id] = next;
      if (Math.round(current) === Math.round(next)) return;
      const newItem = {
        ...x,
        index,
        x: Math.floor(Math.random() * 100),
        y: Math.floor(Math.random() * 100),
        text: `#${Math.round(next)}`,
        rand2: Math.round(Math.random()),
        rand360: Math.round(Math.random() * 720) - 360,
      };
      data = [...data.slice(-99), newItem];
      // console.log(newItem);
    }, 100);
    if (window.already) {
      window.already.process = gProcess;
    }
  });
  onDestroy(() => {
    ded = true;
  });

  const reqMic = async () => {
    if (!window.already) {
      // create audio context
      const audioContext = new AudioContext();
      // request microphone access
      const stream = await navigator.mediaDevices.getUserMedia({
        audio: true,
        video: false,
      });
      // create analyser
      const analyser = audioContext.createAnalyser();
      analyser.fftSize = 1024;
      // connect microphone to analyser
      const microphone = audioContext.createMediaStreamSource(stream);
      microphone.connect(analyser);
      // start analysing
      const data = new Uint8Array(analyser.frequencyBinCount);
      const update = () => {
        analyser.getByteFrequencyData(data);
        window.already.process(data);
        requestAnimationFrame(update);
      };
      window.already = {
        process: gProcess,
      };
      update();
    }
  };
  const gProcess = (data) => {
    // console.log(data);
    const curv = (x) => {
      return 1 - Math.pow(1 - x, 2);
    };
    bV = curv(data[1] / 256);
    cV = curv(data[100] / 256);
    document.body.dispatchEvent(new CustomEvent("mic", { detail: data }));
  };
  let bV = 1;
  let cV = 1;
</script>

<svelte:window on:click={reqMic} />
<div style="--b:{bV};--c:{cV}">
  {#each data as p (p.index)}
    <svg
      width="100"
      height="100"
      viewBox="0 0 100 100"
      xmlns="http://www.w3.org/2000/svg"
      class="f"
      style="position: absolute;
      left: {p.x}%;
      top: {p.y}%;
      --angle: {p.rand360}deg;
  "
    >
      <g transform="translate(50,50)">
        <g class="gg" data-rand2={p.rand2}>
          <g class="w">
            <rect
              x="-50"
              y="-50"
              width="100"
              height="100"
              fill="url(#linear_gradient_{p.id})"
            />
            <!-- add text in the middle of the square -->
            <text
              x="0"
              y="0"
              text-anchor="middle"
              alignment-baseline="middle"
              fill="white"
              font-size="20"
              font-family="Sarabun, sans-serif"
            >
              {p.text}
            </text>
          </g>
        </g>
      </g>
    </svg>
  {/each}
</div>
<Beat />

<style>
  .f {
    animation: x 20s;
    width: 200px;
    height: 200px;
    margin: -100px 0 0 -100px;
    animation-fill-mode: both;
    /* mix-blend-mode: difference; */
  }
  @keyframes x {
    0% {
      opacity: 0;
      transform: rotate(var(--angle)) scale(0.01);
    }
    5% {
      opacity: 1;
      transform: rotate(0deg) scale(2);
    }
    10% {
      opacity: 1;
      transform: rotate(0deg) scale(1);
    }
    100% {
      opacity: 0;
    }
  }
  .w {
    animation: y 1s infinite alternate;
  }
  /* wiggle */
  @keyframes y {
    0% {
      transform: rotate(-5deg);
    }
    100% {
      transform: rotate(5deg);
    }
  }
  .gg {
    scale: var(--b);
  }
  .gg[data-rand2="1"] {
    scale: var(--c);
  }
</style>
