<script>
  // Import necessary libraries
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  // Sample data for the bar chart
  let data = [7, 15, 23, 10, 18, 30, 5, 27, 2, 12, 45, 48, 35, 22, 19, 1, 3];

  // Function to execute after component is mounted
  onMount(() => {
    // Define margins and dimensions for the chart area
    const margin = { top: 20, right: 20, bottom: 40, left: 20 };
    const width = 400 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    // Select the container and append an SVG element
    const svg = d3.select('.chart-container')
      .append('svg')
      .attr('width', '100%')
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    // Create a linear y scale for the data
    const yScale = d3.scaleLinear()
      .domain([0, d3.max(data)]).nice() // Set domain and round to nice values
      .range([height, 0]); // Set range for y-axis

    const barWidth = 25; // Define the width of each bar

    // Bind data to SVG elements and create the bars
    svg.selectAll('.bar')
      .data(data)
      .enter()
      .append('rect')
      .attr('class', 'bar') // Assign the 'bar' class for styling
      .attr('x', (d, i) => i * (barWidth + 5) + 5) // Position the bar horizontally
      .attr('y', d => yScale(d)) // Position the bar vertically based on data
      .attr('width', barWidth) // Set the width of the bar
      .attr('height', d => height - yScale(d)) // Set the height of the bar based on data
      .attr('fill', '#33a0f990'); // Set the fill color of the bars

    // Append y-axis to the chart
    svg.append("g")
      .call(d3.axisLeft(yScale).tickFormat((y) => (y).toFixed())) // Add y-axis with formatted tick labels
      .call(g => g.select(".domain").remove()); // Remove domain line from the y-axis
  });
</script>

<!-- Container for the chart -->
<div class="chart-container"></div>

<style>
    /* Define styles for the chart container */
    .chart-container {
        /*padding-top: 5rem;*/
    }
</style>
