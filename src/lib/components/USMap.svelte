<script>
  // Import necessary Svelte functions and libraries
  import { onMount } from 'svelte';
  import * as topojson from 'topojson-client'; // Import TopoJSON library
  import { geoPath, geoAlbersUsa } from 'd3-geo'; // Import D3's geographic functions
  import { draw } from 'svelte/transition';

  // Set up a geographical projection for the map
  const projection = geoAlbersUsa().scale(1300).translate([487.5, 305])

  // Create a function to generate SVG path data
  const path = geoPath().projection(null);

  // Initialize variables to store map data
  let states = [];
  let mesh;
  let selected;

  // Fetch map data and process it when the component mounts
  onMount(async () => {
                            // Url to map data json
    const us = await fetch('https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json')
      .then(d => d.json()); // Fetch map data from a JSON source

    // Extract features for states
    states = topojson.feature(us, us.objects.states).features;

    // Create a mesh to separate state boundaries
    mesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);
  })
</script>

<!-- Render the map using SVG elements -->
<svg viewBox="0 0 975 610">
    <!-- State shapes -->
    <g fill="white" stroke="black">
        {#each states as feature, i}
            <!-- Render state shapes using path data -->
            <path d={path(feature)} on:click={() => selected = feature} class="state" in:draw={{ delay: i * 50, duration: 1000 }} />
        {/each}
    </g>

    <!-- Highlight selected state -->
    {#if selected}
        <!-- Display a highlighted path for the selected state -->
        <path d={path(selected)} fill="#ecff3150" stroke="black" stroke-width={2} />
    {/if}
</svg>

<!-- Display the name of the selected shape -->
<div class="selectedName">{selected?.properties.name ?? ''}</div>

<!-- Apply styles to the map elements -->
<style>
    svg {
        width: 800px;
    }
    /* Apply a hover effect to the state shapes */
    .state:hover {
        fill: #ecff31;
    }

    /* Style the selectedName element */
    .selectedName {
        text-align: center;
        margin-top: 8px;
        font-size: 1.5rem;
    }
</style>
