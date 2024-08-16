<script>
  import world from "$data/110m.json";
  import * as topojson from "topojson-client";
  import { geoOrthographic, geoPath, geoCentroid } from "d3-geo";
  import Glow from "$components/Glow.svelte";
  import data from "$data/data.json";
  import { map, max } from "d3-array";
  import { scaleLinear } from "d3-scale";
  import { timer } from "d3-timer";
  import { onMount } from "svelte";
  import { select } from "d3-selection";
  import { drag } from "d3-drag";

  import { spring } from "svelte/motion";
  import Tooltip from "$components/Tooltip.svelte";
  import { draw } from "svelte/transition";
  import Legend from "$components/Legend.svelte";

  let xRotation = spring(0, {
    stiffness: 0.2,
    damping: 0.4,
  });

  let yRotation = spring(0, {
    stiffness: 0.3,
    damping: 0.7,
  });

  const degreesPerFrame = 0.5;
  let dragging = false;

  const t = timer(() => {
    if (dragging || tooltipData) return;
    $xRotation += degreesPerFrame;
  }, 1);

  let countries = topojson.feature(world, world.objects.countries).features;
  let borders = topojson.mesh(
    world,
    world.objects.countries,
    (a, b) => a !== b,
  );

  let width = 400;
  $: height = width; // Because it is a circle, and we want it to update anytime width changes

  $: projection = geoOrthographic()
    .scale(width / 2)
    .rotate([$xRotation, $yRotation])
    .translate([width / 2, height / 2]);

  $: path = geoPath(projection);

  const colorScale = scaleLinear()
    .domain([0, max(data, (d) => d.population)])
    .range(["#26362e", "#0DCC6C"]);

  countries.forEach((country) => {
    const metadata = data?.find((d) => d.id === country.id);
    if (metadata) {
      country.population = metadata.population;
      country.country = metadata.country;
    }
  });

  let globe;
  const dragSensitivity = 0.3;

  onMount(() => {
    const element = select(globe);
    element.call(
      drag()
        .on("drag", (event) => {
          dragging = true;
          $xRotation = $xRotation + event.dx * dragSensitivity;
          $yRotation = $yRotation - event.dy * dragSensitivity;
        })
        .on("end", () => {
          dragging = false;
        }),
    );
  });

  let tooltipData;

  // Whenever tooltipData changes, calculate the center of the country and rotate to it
  $: if (tooltipData) {
    const center = geoCentroid(tooltipData);
    $xRotation = -center[0];
    $yRotation = -center[1];
  }
</script>

<div class="chart-container" bind:clientWidth={width}>
  <h1>Population by Country</h1>
  <h2>Click on a country to see its population. Click on the sea to get back.</h2>
  <svg {width} {height} bind:this={globe} class:dragging>
    <!-- Glow -->
    <Glow></Glow>

    <!-- Globe -->
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <circle
      r={width / 2}
      cx={width / 2}
      cy={height / 2}
      fill="#1C1C1C"
      filter="url(#glow)"
      on:click={() => {
        tooltipData = null;
      }}
    />

    <!-- Countries -->
    {#each countries.sort((a,b) => b.population - a.population) as country}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
      <path
        d={path(country)}
        fill={colorScale(country.population || 0)}
        stroke="none"
        on:click={() => {
          tooltipData = country;
        }}
        on:focus={() => (tooltipData = country)}
        tabindex="0"
      />
    {/each}

    <!-- Borders -->
    <path d={path(borders)} fill="none" stroke="black" />

    <!-- Borders of selected country-->
    {#if tooltipData}
      {#key tooltipData.id}
        <path
          d={path(tooltipData)}
          fill="transparent"
          stroke="white"
          stroke-width="2"
          pointer-events="none"
          in:draw
        />
      {/key}
    {/if}
  </svg>
  <Tooltip data={tooltipData}></Tooltip>
  <Legend data={tooltipData} {colorScale} ></Legend>
</div>

<style>
  .chart-container {
    max-width: 668px;
    margin: 0 auto;
  }

  :global(body) {
    background: rgb(40, 40, 40);
  }

  svg {
    margin-top: 20px;
    margin-bottom: 10px;
    overflow: visible;
  }

  .dragging {
    cursor: grabbing;
  }

  path {
    cursor: pointer;
  }

  path:focus {
    outline: none;
  }

  h1,
h2 {
  color: white;
  text-align: center;
}

h1 {
  font-size: 1.75rem;
  font-weight: 600;
  margin-top: 1rem;
  margin-bottom: 0.8rem;
}

h2 {
  font-size: 1.25rem;
  font-weight: 200;
  margin-bottom: .5rem;
}
</style>
