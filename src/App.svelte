<script lang="ts">
import BarChart from "./lib/components/BarChart.svelte";
import InteractiveBarChart from "./lib/components/InteractiveBarChart.svelte";
import USMap from "./lib/components/USMap.svelte";
import AnimatedBarChart from "./lib/components/AnimatedBarChart.svelte";
import charts from "./assets/charts.json"
import WorldGlobe from "./lib/components/WorldGlobe.svelte";
import BubbleChart from "./lib/components/BubbleChart.svelte";

let activeChart = 'Bar chart'
const handleClick = (chart: string) => {
  activeChart = chart
}

</script>

<main>
  <div class="container">
    <div class="navigation">
      <ul>
        {#each charts as chart}
          <li class:active={activeChart === chart.link} on:click={() => handleClick(chart.link)}>{chart.link}</li>
        {/each}
      </ul>
    </div>
    <div class="chart">
      <h2>{charts.find(chart => chart.link === activeChart).title}</h2>
      <p>{charts.find(chart => chart.link === activeChart).description}</p>
      {#if activeChart === 'Bar chart'}
        <BarChart/>
      {:else if activeChart === 'Interactive Bar chart'}
        <InteractiveBarChart/>
      {:else if activeChart === 'Animated Bar chart'}
        <AnimatedBarChart/>
      {:else if activeChart === 'Bubble chart'}
        <BubbleChart/>
      {:else if activeChart === 'Map'}
        <USMap/>
      {:else if activeChart === 'World Globe'}
        <WorldGlobe/>
      {/if}
    </div>
  </div>
</main>

<style>
  .container {
    display: flex;
    flex-direction: column;
    justify-content: center; /* Horizontally center */
    align-items: center; /* Vertically center */
  }
  .navigation {
    position: fixed;
    top: 0;
    background-color: #242424;
  }

  .chart {
    padding: 2.5rem 0;
    width: 800px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: row;
    gap: 1rem;
  }

  li {
    padding: 1rem;
    color: #aeaeae;
    transition: color 0.3s;
  }

  li:hover {
    cursor: pointer;
    color: #ffffff;
  }

  li.active {
    color: #ecff31;
  }

  h1 {
    color: #aeaeae;
  }

  p {
    color: #aeaeae;
    padding-bottom: 2.5rem;
  }
</style>
