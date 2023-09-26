<script>
  import { onMount, onDestroy } from 'svelte';
  import * as d3 from 'd3';

  // Import data from the JSON file
  import data from '../../assets/data.json';

  // Initialize variables
  let svgContainer = null;
  let circles = []; // Store the circle data
  let sliderValue = 1800;
  let isPlaying = false;
  let updateInterval = null; // Initialize the interval as null

  // Define margins, height, and width
  const margin = { top: 20, right: 20, bottom: 35, left: 40 };
  const height = 400;
  const width = 800;

  // Define functions and scales
  const bisectDate = d3.bisector(([date]) => date).left;
  const radius = d3.scaleSqrt([0, 5e8], [0, width / 24]);
  const interval = d3.utcMonth;

  function parseSeries(series) {
    return series.map(([dateString, value]) => [new Date(dateString).getFullYear(), value]);
  }

  // Parse the data and extract required information
  const countriesData = data.map(({ name, region, income, population, lifeExpectancy }) => ({
    name,
    region,
    income: parseSeries(income),
    population: parseSeries(population),
    lifeExpectancy: parseSeries(lifeExpectancy)
  }));

  // Define color scale based on regions
  const color = d3.scaleOrdinal(countriesData.map(d => d.region), d3.schemeCategory10).unknown('black');

  // Define x and y axes and scales
  const xAxis = g =>
    g.attr('transform', `translate(0,${height - margin.bottom})`).call(d3.axisBottom(x).ticks(width / 80, ',')).call(g => g.select('.domain').remove()).call(g =>
      g.append('text')
        .attr('x', width)
        .attr('y', margin.bottom - 4)
        .attr('fill', 'currentColor')
        .attr('text-anchor', 'end')
        .text('Income per capita (dollars) →')
    );
  const x = d3.scaleLog([200, 1e5], [margin.left, width - margin.right]);

  const yAxis = g =>
    g
      .attr('transform', `translate(${margin.left},0)`)
      .call(d3.axisLeft(y))
      .call(g => g.select('.domain').remove())
      .call(g =>
        g
          .append('text')
          .attr('x', -margin.left)
          .attr('y', 10)
          .attr('fill', 'currentColor')
          .attr('text-anchor', 'start')
          .text('↑ Life expectancy (years)')
      );
  const y = d3.scaleLinear([14, 86], [height - margin.bottom, margin.top]);

  // Execute code when component is mounted
  onMount(() => {

    // Define grid lines
    const grid = g =>
      g.attr('stroke', 'currentColor').attr('stroke-opacity', 0.1).call(g =>
        g
          .append('g')
          .selectAll('line')
          .data(x.ticks())
          .join('line')
          .attr('x1', d => 0.5 + x(d))
          .attr('x2', d => 0.5 + x(d))
          .attr('y1', margin.top)
          .attr('y2', height - margin.bottom)
      ).call(g =>
        g
          .append('g')
          .selectAll('line')
          .data(y.ticks())
          .join('line')
          .attr('y1', d => 0.5 + y(d))
          .attr('y2', d => 0.5 + y(d))
          .attr('x1', margin.left)
          .attr('x2', width - margin.right)
      );

    // Create SVG container
    svgContainer = d3.select('.chart-container')
      .append('svg')
      .attr('viewBox', [0, 0, width, height]);

    // Append axes and grid
    svgContainer.append('g').call(xAxis);
    svgContainer.append('g').call(yAxis);
    svgContainer.append('g').call(grid);

    // Update initial circles
    updateCircles();

    // Return the SVG container node
    return Object.assign(svgContainer.node(), {});
  });

  // Function to fetch data at a specific date
  function dataAt(date) {
    return countriesData.map(d => ({
      name: d.name,
      region: d.region,
      income: valueAt(d.income, date),
      population: valueAt(d.population, date),
      lifeExpectancy: valueAt(d.lifeExpectancy, date)
    }));
  }

  // Function to interpolate values between two dates
  function valueAt(values, date) {
    const i = bisectDate(values, date, 0, values.length - 1);
    const a = values[i];
    if (i > 0) {
      const b = values[i - 1];
      const t = (date - a[0]) / (b[0] - a[0]);
      return a[1] * (1 - t) + b[1] * t;
    }
    return a[1];
  }

  // Function to handle slider value change
  function handleSliderChange(event) {
    sliderValue = event.target.value;
    updateCircles();
  }

  // Function to toggle play/pause animation
  function togglePlay() {
    if (updateInterval !== null) { // Check if the animation is already running
      clearInterval(updateInterval);
      updateInterval = null; // Reset the interval
      isPlaying = false; // Animation is paused
    } else {
      updateCircles(); // Initial circles update before starting animation
      if (sliderValue >= 2005) {
        sliderValue = 1800; // Reset the slider value if it's at or beyond 2005
      }
      updateInterval = setInterval(() => {
        sliderValue += 1;
        if (sliderValue >= 2005) {
          clearInterval(updateInterval); // Stop the interval when reaching 2005
          updateInterval = null; // Reset the interval
          isPlaying = false; // Animation is stopped
        } else {
          updateCircles(); // Continue animation
        }
      }, 100);
      isPlaying = true; // Animation is playing
    }
  }

  // Function to update circles on the chart
  function updateCircles() {
    if (svgContainer) {
      // Remove existing circles
      svgContainer.selectAll('circle').remove();

      // Update circle data based on current slider value
      circles = dataAt(sliderValue).map(d => ({
        data: d,
        circle: svgContainer.append('circle')
      }));

      // Create circles and set attributes
      circles.forEach(({ data, circle }) => {
        circle
          .sort((a, b) => d3.descending(a.data.population, b.data.population))
          .attr('cx', x(data.income))
          .attr('cy', y(data.lifeExpectancy))
          .attr('r', radius(data.population))
          .attr('fill', color(data.region))
          .call(circle =>
            circle
              .append('title')
              .text(d => [data.name, data.region].join('\n'))
          );
      });
    }
  }

  // Execute code when component is destroyed
  onDestroy(() => {
    clearInterval(updateInterval); // Clear the animation interval
  });
</script>

<!-- Play/Pause button, slider, and slider value display -->
<div>
    <button on:click={togglePlay}>{isPlaying ? 'Pause' : 'Play'}</button>
    <input
            type="range"
            min=1800
            max=2005
            step=1
            bind:value={sliderValue}
            on:input={handleSliderChange}
    />
    <span>{sliderValue}</span>
</div>

<!-- Chart container for the SVG visualization -->
<div class="chart-container" bind:this={svgContainer}></div>

<!-- Legend to display color meanings for regions -->
<div class="legend">
    {#each color.domain() as region}
        <div class="legend-item">
            <span class="legend-color" style="background-color: {color(region)};"></span>
            <span class="legend-label">{region}</span>
        </div>
    {/each}
</div>

<style>
    /* Add CSS styling here */

    /* Styling for the play/pause button, slider, and value display */
    button {
        margin-bottom: 1rem;
    }

    /* Styling for the legend */
    .legend {
        display: flex;
        justify-content: center;
        align-items: center;
        margin-top: 2rem;
    }

    .legend-item {
        display: flex;
        align-items: center;
        margin-right: 1.5rem;
    }

    .legend-color {
        display: inline-block;
        width: 0.875rem;
        height: 0.875rem;
        margin-right: 0.25rem;
    }

    .legend-label {
        font-size: 10px;
    }
</style>
