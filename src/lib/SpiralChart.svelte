<script>
  export let data;

  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  // handle git log data
  const timeParse = d3.timeParse('%a %b %-d %I:%M:%S %Y');
  const dateFormat = d3.timeFormat('%Y-%m-%d');
  const dateParse = d3.timeParse('%Y-%m-%d');

  const gitLog = d3.rollup(
    data.map((d) => {
      d.date = timeParse(d.date);
      d.date = dateFormat(d.date);
      d.date = dateParse(d.date);
      return d;
    }),
    (v) => v.length,
    (d) => d.date
  );

  // set chart dimension
  let width = 800;
  $: height = width * 0.8;
  let spirals = 2.05;
  let spiralSpace = 30;
  $: r = width / 2 - spiralSpace * 4;
  $: scaleRadius = d3.scaleLinear().domain([0, spirals]).range([spiralSpace, r]);
  $: spiralPoints = d3.range(0, spirals + 0.001, spirals / 100);
  $: spiralPath = d3
    .lineRadial()
    .angle((r) => Math.PI * r * spirals)
    .radius(scaleRadius)
    .curve(d3.curveCardinal);

  let spiralElm;
  $: spiralLength = spiralElm && spiralElm.getTotalLength();
  $: dateScale = spiralLength && d3.scaleTime().domain(d3.extent(gitLog.keys())).range([0, spiralLength]);

  $: getCommitsPos = (date) => {
    return spiralElm.getPointAtLength(dateScale(date));
  };

  // bubble chart
  $: bubbleRadius = d3
    .scaleLinear()
    .domain(d3.extent([...gitLog].map((d) => d[1])))
    .range([1, 48]);

  // bar chart
  $: barWidth = spiralLength && spiralLength / gitLog.size - 2;
  $: barHieght = d3
    .scaleLinear()
    .domain(d3.extent([...gitLog].map((d) => d[1])))
    .range([1, 100]);
  $: getCommitsAnglePos = (date) => {
    return spiralElm.getPointAtLength(dateScale(date) - barWidth);
  };

  // current progress
  let gitLogArray = [...gitLog].sort((a, b) => d3.ascending(a[0], b[0]));
  let currentDate = d3.min(gitLog.keys());
  let minDate = d3.min(gitLog.keys());
  let maxDate = d3.max(gitLog.keys());
  let currentPos;
  let i = 20;
  $: dateRange = gitLogArray.map((d) => d[0]);
  $: totalCommits = d3.sum(
    gitLogArray.filter((d, id) => id <= i),
    (d) => d[1]
  );
  let animationStart = false;
  let animationDone = false;

  // animation
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

    observer.observe(document.querySelector('.spiral'));
  });

  $: if (animationStart) {
    mainAnimation();
  }

  const mainAnimation = () => {
    // avatar animation
    const timer = setInterval(() => {
      currentDate = dateRange[i];
      if (i >= dateRange.length - 1) {
        clearInterval(timer);
        setTimeout(() => {
          animationDone = true;
        }, 200);
      }
      currentPos = getCommitsPos(currentDate);
      i += 5;
    }, 50);

    // bars animation
    d3.selectAll('#spiral-bars > rect')
      .transition()
      .delay(function (d, i) {
        return i * parseInt(4860 / 486);
      })
      .attr('opacity', 1)
      .attr('height', (d, i) => {
        return barHieght(gitLogArray[i][1]);
      });
  };
</script>

<style lang="scss">
  .spiral-wrap {
    width: 100%;
    max-width: 800px;
    margin: 0 auto;
  }

  .spiral {
    position: relative;

    > img {
      position: absolute;
      transition: all 0.1s linear;
    }
  }

  h2 {
    text-align: center;
    font-family: 'Noto Sans TC';
    color: #2d2d2d;
    margin-bottom: 32px;
  }
</style>

<h2>Daniel 入職以來，共提交了<span style:color="purple"> {animationStart ? totalCommits : '？'} </span>次commit！</h2>
<div class="spiral-wrap" style:height={height + 'px'} bind:clientWidth={width}>
  <div class="spiral">
    <svg viewBox={`${-width / 2} ${-height / 2} ${width} ${height}`}>
      <path bind:this={spiralElm} d={spiralPath(spiralPoints)} stroke="none" fill="none" />
      <g id="spiral-bars">
        {#if dateScale}
          {#each gitLogArray as [key]}
            {@const pos = getCommitsPos(key)}
            {@const aPos = getCommitsAnglePos(key)}
            {@const x = pos.x}
            {@const y = pos.y}
            {@const angle = (Math.atan2(aPos.y, aPos.x) * 180) / Math.PI - 90}
            <rect
              opacity="0"
              {x}
              {y}
              width={barWidth}
              fill="purple"
              height="0"
              transform={`rotate(${angle} ${x} ${y})`}
            />
            <!-- <circle cx={pos.x} cy={pos.y} r={bubbleRadius(value)} stroke="#216E39" fill="#216E39" fill-opacity="0.2" /> -->
          {/each}
        {/if}
      </g>
      {#if dateScale}
        <text
          x={getCommitsPos(minDate).x}
          y={getCommitsPos(minDate).y + 20}
          color="#2d2d2d"
          text-anchor="middle"
          font-size="14">2020/3</text
        >
        {#if animationDone}
          <text
            x={getCommitsPos(maxDate).x}
            y={getCommitsPos(maxDate).y + 60}
            color="#2d2d2d"
            text-anchor="middle"
            font-size="14">2022/2</text
          >
        {/if}
      {/if}
    </svg>
    {#if currentPos && dateScale}
      <img
        width="80"
        height="80"
        style:top={currentPos.y - 40 + height / 2 + 'px'}
        style:left={currentPos.x - 40 + width / 2 + 'px'}
        src="/daniel-avatar.png"
        alt="Daniel!"
      />
    {/if}
  </div>
</div>
