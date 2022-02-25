<script>
  import { fade } from 'svelte/transition'
  import { onMount } from 'svelte'

  export let data


  let showContent = false
  onMount(() => {
    let observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].intersectionRatio > 0.5 && showContent === false) {
          showContent = true;
        }
      },
      {
        root: null,
        threshold: [0.5, 0.75, 1],
      }
    );

    observer.observe(document.querySelector('#we-love-daniel'));
  });

</script>

<style>
  .wrap {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin-bottom: 200px;
  }

  p {
    font-size: 24px;
    font-family: 'Noto Sans TC';
    margin: 16px;
    width: 600px;
  }
</style>

<div class="wrap" id="we-love-daniel">
  <p style:text-align="center" style:font-size="24px">大家想對尼爾說的話</p>
  {#if showContent}
    {#each data as {name, content}, i}
      <p in:fade={{delay: 500 + i * 1200}}>{name + '：' + content}</p>
    {/each}
  {/if}
</div>