<script>
  export let data;

  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let width = 800;
  $: height = width;

  // generate fake data
  const pData = data.map((d) => {
    d.value = d3.randomInt(50, 200)();
    return d;
  });
  $: pack = (data) => d3.pack().size([width, height]).padding(10)(d3.hierarchy({ children: data }).sum((d) => d.value));

  $: bubbleData = pack(pData).leaves();

  let animationStart = false;
  onMount(() => {
    let observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].intersectionRatio > 0.5 && animationStart === false) {
          animationStart = true;
        }
      },
      {
        root: null,
        threshold: [0.5, 0.75, 1],
      }
    );

    observer.observe(document.querySelector('.bubble'));
  });

  $: if (animationStart) {
    d3.selectAll('#bubbles > circle')
      .data(bubbleData)
      .transition()
      .delay((d, i) => i * 50)
      .attr('r', (d) => d.r);
  }
</script>

<style>
  .bubble-wrap {
    width: 100%;
    max-width: 800px;
    min-height: 100vh;
    margin: 0 auto;
  }

  .bubble {
    position: relative;
  }

  h2 {
    text-align: center;
    font-family: 'Noto Sans TC';
    color: #2d2d2d;
    margin-bottom: 0;
  }
</style>

<h2>Daniel 做的專題有這麽多人看過！</h2>
<div class="bubble-wrap">
  <div class="bubble">
    <svg viewBox={`0 0 ${width} ${height}`}>
      <g id="bubbles" transform="translate(0, 20)">
        {#each bubbleData as { r, x, y }}
          <circle cx={x} cy={y} r="0" fill="#EB868C" />
        {/each}
      </g>
    </svg>
  </div>
</div>
