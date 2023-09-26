<script>
  // Import necessary libraries
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  // Sample data for the bar chart
  let data = [
    {letter: "A", frequency: 0.08167},
    {letter: "B", frequency: 0.01492},
    {letter: "C", frequency: 0.02782},
    {letter: "D", frequency: 0.04253},
    {letter: "E", frequency: 0.12702},
    {letter: "F", frequency: 0.02288},
    {letter: "G", frequency: 0.02015},
    {letter: "H", frequency: 0.06094},
    {letter: "I", frequency: 0.06966},
    {letter: "J", frequency: 0.00153},
    {letter: "K", frequency: 0.00772},
    {letter: "L", frequency: 0.04025},
    {letter: "M", frequency: 0.02406},
    {letter: "N", frequency: 0.06749},
    {letter: "O", frequency: 0.07507},
    {letter: "P", frequency: 0.01929},
    {letter: "Q", frequency: 0.00095},
    {letter: "R", frequency: 0.05987},
    {letter: "S", frequency: 0.06327},
    {letter: "T", frequency: 0.09056},
  ];
  let orderOption = "0"; // Default order option

  // Function to execute after component is mounted
  onMount(() => {
    // Define margins and dimensions for the chart area
    const margin = { top: 20, right: 0, bottom: 40, left: 20 };
    const width = 650 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    // Select the container and append an SVG element
    const svg = d3.select('.chart-container')
      .append('svg')
      .attr('width', '100%')
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    // Declare the x (horizontal position) scale and the corresponding axis generator.
    const xScale = d3.scaleBand()
      .domain(data.map(d => d.letter))
      .range([0, width - margin.right])
      .padding(0.1);

    const xAxis = d3.axisBottom(xScale).tickSizeOuter(0);

    // Create a linear y scale for the data
    const yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.frequency)]).nice() // Set domain and round to nice values
      .range([height - margin.bottom, 0]); // Set range for y-axis

    const barWidth = 25; // Define the width of each bar

    // Bind data to SVG elements and create the bars
    svg.selectAll('.bar')
      .data(data)
      .enter()
      .append('rect')
      .attr('class', 'bar') // Assign the 'bar' class for styling
      .attr('x', d => xScale(d.letter)) // Position the bar horizontally
      .attr('y', d => yScale(d.frequency)) // Position the bar vertically based on data
      .attr('width', barWidth) // Set the width of the bar
      .attr('height', d => yScale(0) - yScale(d.frequency)) // Set the height of the bar based on data
      .attr('fill', '#33a0f9') // Set the fill color of the bars
      .on('mouseover', function (event, d) {
        d3.select(this)
          .attr('fill', '#e08c5c') // Change color on hover
          .attr('cursor', 'pointer'); // Change cursor to pointer

        // Show value as tooltip
        const tooltip = svg
          .append('text')
          .attr('class', 'tooltip')
          .attr('x', parseFloat(this.getAttribute('x')) + barWidth / 2)
          .attr('y', parseFloat(this.getAttribute('y')) - 5)
          .attr('text-anchor', 'middle')
          .style('fill', '#ffffff')
          .text((d.frequency * 100).toFixed(2));
      })
      .on('mouseout', function () {
        d3.select(this).attr('fill', '#33a0f9'); // Restore original color

        // Remove tooltip on mouseout
        svg.select('.tooltip').remove();
      });

    // Create the axes.
    const gx = svg.append("g")
      .attr("transform", `translate(0,${height - margin.bottom})`)
      .call(xAxis)
      .call(g => g.select(".domain").remove());

    const gy = svg.append("g")
      .attr("transform", `translate(0,0)`)
      .call(d3.axisLeft(yScale).tickFormat((y) => (y * 100).toFixed()))
      .call(g => g.select(".domain").remove());

    // Function to update the chart based on the selected order
    const updateChart = () => {
      let orderedData = [...data]; // Clone the original data

      // Sort the data based on the selected order option
      if (orderOption === "0") {
        orderedData.sort((a, b) => a.letter.localeCompare(b.letter)); // Alphabetical
      } else if (orderOption === "1") {
        orderedData.sort((a, b) => a.frequency - b.frequency); // Frequency, ascending
      } else if (orderOption === "2") {
        orderedData.sort((a, b) => b.frequency - a.frequency); // Frequency, descending
      }

      // Update xScale domain based on the ordered data
      xScale.domain(orderedData.map(d => d.letter));

      // Update the x position of the bars based on the new order
      svg.selectAll('.bar')
        .transition()
        .duration(500) // Add a transition for a smooth update
        .attr('x', d => xScale(d.letter));

      // Update the x-axis labels
      gx.call(xAxis);
    };

    // Call the updateChart function initially
    updateChart();

    // Event listener for select tag changes
    const selectElement = document.querySelector(".select-order");
    selectElement.addEventListener("change", () => {
      orderOption = selectElement.value; // Update the selected order option
      updateChart(); // Call the function to update the chart
    });
  });
</script>

<label>Select bar chart order</label>
<select class="select-order" name="select">
    <option value="0">Alphabetical</option>
    <option value="1">Frequency, ascending</option>
    <option value="2">Frequency, descending</option>
</select>
<!-- Container for the chart -->
<div class="chart-container"></div>

<style>
    /* Define styles for the chart container */
    .chart-container {
        /*padding-top: 5rem;*/
    }
    .select-order {
        width: 200px;
    }

    label {
        font-size: 0.9rem;
        color: #aeaeae;
        padding-bottom: 0.25rem;
    }

    select {
        /*font-size: 1rem;*/
        background-color: transparent;
        /*border: none;*/
        padding: 0.5rem 0.25rem;
        outline: none;
        border-radius: 0.25rem;
        -webkit-box-shadow: none;
        -moz-box-shadow: none;

        &:hover {
            cursor: pointer;
        }
    }
</style>
