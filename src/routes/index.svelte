<script context="module">
  import { csvParse } from 'd3';

  export async function load({ fetch }){
    const gitRes = await fetch('data/repo-logs-min.json')
    const bubbleRes = await fetch('data/projects.csv')
    const contentRes = await fetch('data/content.csv')


    return {
      status: gitRes.status,
      props: {
        gitLog: gitRes.ok && (await gitRes.json()),
        bubbleData: bubbleRes.ok && csvParse(await bubbleRes.text()),
        contentData: contentRes.ok && csvParse(await contentRes.text())
      }
    }
  }
</script>


<script>
  export let gitLog
  export let bubbleData
  export let contentData

  import SpiralChart from '$lib/SpiralChart.svelte';
  import BubbleChart from '$lib/BubbleChart.svelte';
  import WeLoveDaniel from '$lib/WeLoveDaniel.svelte';

  let windowWidth

</script>

<style lang="scss">
  main {
    max-width: 1200px;
    padding: 24px;
    margin: 0 auto;
  }

  .intro {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    h1 {
      text-align: center;
      font-family: 'Noto Sans TC';
      color: #2d2d2d;
      margin-bottom: 80px;
    }

    #hand {
      font-size: 28px;
    }

    .wave {
      animation-name: wave-animation;
      animation-duration: 2.5s;
      animation-iteration-count: infinite;
      transform-origin: 70% 70%;
      display: inline-block;
    }

    @keyframes wave-animation {
      0% {
        transform: rotate(0deg);
      }
      10% {
        transform: rotate(14deg);
      }
      20% {
        transform: rotate(-8deg);
      }
      30% {
        transform: rotate(14deg);
      }
      40% {
        transform: rotate(-4deg);
      }
      50% {
        transform: rotate(10deg);
      }
      60% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(0deg);
      }
    }
  }

  section {
    margin-bottom: 200px;
  }
</style>

<svelte:window bind:innerWidth={windowWidth} />

<main>

{#if windowWidth < 768}
   <p style:text-align="center">如果你夠喜歡 Daniel 就打開電腦看❤️</p>
{:else}
  <section class="intro">
    <h1>Daniel Kao <span class="wave" id="hand">👋</span><span class="wave" id="hand">👋</span></h1>
    <img width="600" src="./泥而.svg" alt="daniel">
  </section>
  <section>
    <BubbleChart data={bubbleData} />
  </section>
  <section>
    <SpiralChart data={gitLog} />
  </section>
  <section>
    <WeLoveDaniel data={contentData} />
  </section>
{/if}
</main>
