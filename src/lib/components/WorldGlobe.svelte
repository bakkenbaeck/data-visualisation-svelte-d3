<script>
  // Import necessary Svelte functions and libraries
  import {afterUpdate, onMount} from 'svelte';
  import { select } from 'd3-selection';
  import { geoOrthographic, geoPath } from 'd3-geo';
  import { drag } from 'd3-drag';
  import countriesData from '../../assets/land50.json';

  // Sample city data with addresses
  const cityData = [
    {
      name: 'BAKKEN & BAECK GMBH',
      coordinates: [7.100000, 50.733334],
      address: 'FÜRSTENSTRASSE 2-4',
      city: '53111 BONN',
      country: 'GERMANY'
    },
    {
      name: 'BAKKEN & BAECK AS',
      coordinates: [10.757933, 59.911491],
      address: 'TRONDHEIMSVEIEN 135',
      city: '0570 OSLO',
      country: 'NORWAY'
    },
    {
      name: 'BAKKEN & BÆCK B.V.',
      coordinates: [4.889070, 52.390160],
      address: 'VAN DIEMENSTRAAT 38',
      city: '1013NH AMSTERDAM',
      country: 'THE NETHERLANDS'
    },
    {
      name: 'BAKKEN & BÆCK S.L.',
      coordinates: [2.193290, 41.395120],
      address: 'C. DE PAMPLONA 59',
      city: '08005 BARCELONA',
      country: 'SPAIN'
    },
    {
      name: 'BAKKEN & BAECK LTD',
      coordinates: [-0.080070, 51.542590],
      address: '23 ENGLEFIELD RD',
      city: 'LONDON N1 4JX',
      country: 'UNITED KINGDOM'
    },
  ];

  let svg;
  let projection;
  let path;
  let isDragging = false;
  let rotation = [0, 0];
  let selectedCity = null; // To store selected city data
  let popupStyle = 'display: none;'; // To store selected city data

  // Function to handle city circle click
  function handleCityClick(event, data) {
    selectedCity = {
      city: data,
      cursorPosition: { x: event.clientX, y: event.clientY },
    };
    if (selectedCity) {
      popupStyle = `display: block; top: ${selectedCity.cursorPosition.y}px; left: ${selectedCity.cursorPosition.x}px;`;
    } else {
      popupStyle = 'display: none;';
    }
  }

  // Function to close the city popup when clicking outside of it
  function handleBodyClick(event) {
    if (selectedCity && !event.target.closest('.city')) {
      selectedCity = null;
      popupStyle = 'display: none;';
    }
  }

  // Function to handle dragging the globe
  function handleDrag(event) {
    selectedCity = null;
    popupStyle = 'display: none;';
    // Check if dragging and data is available
    if (isDragging && countriesData) {
      const sensitivity = 0.2;
      // Update rotation based on drag movement
      rotation[0] += event.dx * sensitivity;
      rotation[1] -= event.dy * sensitivity;
      projection.rotate(rotation);
      // Update country paths with new rotation
      svg.selectAll('path.country').attr('d', path);
      // Update city dots with new projection
      svg.selectAll('circle.city')
        .attr('cx', d => projection(d.coordinates)[0])
        .attr('cy', d => projection(d.coordinates)[1]);
    }
  }

  // On component mount, initialize the globe and SVG element
  onMount(async () => {
    const width = 800;
    const height = 600;

    // Initialize orthographic projection
    projection = geoOrthographic()
      .fitSize([width, height], { type: 'Sphere' })
      .translate([width / 2, height / 2])
      .rotate(rotation);

    // Create path generator
    path = geoPath().projection(projection);

    // Select and set up SVG element
    svg = select('svg')
      .attr('width', width)
      .attr('height', height)
      .attr('cursor', 'grabbing')
      .call(
        drag()
          .on('start', () => {
            isDragging = true;
          })
          .on('drag', handleDrag)
          .on('end', () => {
            isDragging = false;
          })
      );

    // Append country paths using data
    if (countriesData) {
      svg.selectAll('path.country')
        .data(countriesData.features)
        .enter()
        .append('path')
        .attr('class', 'country')
        .attr('fill', '#aeaeae')
        .attr('stroke', '#ecff31')
        .attr('stroke-width', '0.5px')
        .attr('d', path);
    }

    // Append city dots using cityData
    svg.selectAll('circle.city')
      .data(cityData)
      .enter()
      .append('circle')
      .attr('cursor', 'pointer')
      .attr('class', 'city')
      .attr('cx', (d) => projection(d.coordinates)[0])
      .attr('cy', (d) => projection(d.coordinates)[1])
      .attr('r', 5)
      .attr('fill', 'red')
      .on('click', (d, event) => {
        handleCityClick(d, event);
      });
  });

  // Add a click event listener to the document body to close the city popup
  onMount(() => {
    document.body.addEventListener('click', handleBodyClick);
  });

  // Cleanup: remove the click event listener when the component is destroyed
  afterUpdate(() => {
    return () => {
      document.body.removeEventListener('click', handleBodyClick);
    };
  });
</script>

<svg bind:this={svg}></svg>
<!-- Add a div for the city popup -->
<div class="city-popup" style={popupStyle}>
    {#if selectedCity}
        <p>{selectedCity.city.name}</p>
        <p>{selectedCity.city.address}</p>
        <p>{selectedCity.city.city}</p>
        <p>{selectedCity.city.country}</p>
    {/if}
</div>

<style>
    .city-popup {
        position: absolute;
        background-color: #ecff31;
        color: #242424;
        padding: 1rem 2rem;
        border-radius: 1rem;
    }
    p {
        padding: 0;
        margin: 0;
    }
</style>
