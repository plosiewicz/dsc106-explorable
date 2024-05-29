<!-- LineGraph.svelte -->
<script>
  import { onMount } from 'svelte';
  import { select } from 'd3-selection';
  import { scaleLinear } from 'd3-scale';
  import { axisBottom, axisLeft } from 'd3-axis';
  import { line } from 'd3-shape';
  import { range } from 'd3-array';

  let fiberIntake = 25;
  let width = 800;
  let height = 400;
  const margin = { top: 50, right: 50, bottom: 70, left: 70 };
  const chartTitle = "Fiber Intake vs Weight Loss in Kg";

  const predictWeightLoss = fiberIntake => fiberIntake * (-0.98 / 5);

  function updateGraph() {
    const svg = select("#graph");

    const xScale = scaleLinear()
      .domain([0, 50])
      .range([margin.left, width - margin.right]);

    const yScale = scaleLinear()
      .domain([-5, 0])
      .range([height - margin.bottom, margin.top]);

    const lineGenerator = line()
      .x(d => xScale(d))
      .y(d => yScale(predictWeightLoss(d)));

    svg.select(".line-path")
      .attr("d", lineGenerator(range(0, 51)));

    const yIntersect = yScale(0); // y-coordinate where the line intersects the x-axis

    svg.select(".vertical-line")
      .attr("x1", xScale(fiberIntake))
      .attr("y1", yIntersect)
      .attr("x2", xScale(fiberIntake))
      .attr("y2", yScale(predictWeightLoss(fiberIntake)));

    // Update axis labels
    svg.select(".x-axis-label")
      .attr("transform", `translate(${width / 2}, ${height - margin.bottom / 3})`)
      .text("Daily Dietary Fiber Intake (g)");

    svg.select(".y-axis-label")
      .attr("transform", `translate(${margin.left / 4}, ${height / 2}) rotate(-90)`)
      .text("Weight Loss (Kg)");
  }

  onMount(() => {
    const svg = select("#graph")
      .attr("width", width)
      .attr("height", height);

    const xScale = scaleLinear()
      .domain([0, 50])
      .range([margin.left, width - margin.right]);

    const yScale = scaleLinear()
      .domain([-5, 0])
      .range([height - margin.bottom, margin.top]);

    svg.append("g")
      .attr("class", "x-axis")
      .attr("transform", `translate(0,${height - margin.bottom})`)
      .call(axisBottom(xScale).tickFormat(d => d + "g"));

    svg.append("text")
      .attr("class", "x-axis-label")
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "hanging")
      .attr("fill", "black");

    svg.append("g")
      .attr("class", "y-axis")
      .attr("transform", `translate(${margin.left},0)`)
      .call(axisLeft(yScale));

    svg.append("text")
      .attr("class", "y-axis-label")
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("fill", "black");

    const lineGroup = svg.append("g").attr("class", "line-group");

    const lineGenerator = line()
      .x(d => xScale(d))
      .y(d => yScale(predictWeightLoss(d)));

    lineGroup.append("path")
      .attr("class", "line-path")
      .attr("fill", "none")
      .attr("stroke", "blue")
      .attr("stroke-width", 2)
      .attr("d", lineGenerator(range(0, 51)));

    const verticalLineGroup = svg.append("g").attr("class", "vertical-line-group");

    verticalLineGroup.append("line")
      .attr("class", "vertical-line")
      .attr("stroke", "red")
      .attr("stroke-width", 2)
      .attr("stroke-dasharray", "5,5");

    updateGraph();
  });

  function handleSliderInput(event) {
    fiberIntake = +event.target.value;
    updateGraph();
  }
</script>

<div class="container">
  <h2 class="chart-title">{chartTitle}</h2>
  <div class="slider-container">
    <input type="range" min="0" max="50" value={fiberIntake} class="slider" on:input={handleSliderInput} />
    <label for="slider">Daily Dietary Fiber Intake (g): <span>{fiberIntake}</span></label>
  </div>
  <div id="graph-container">
    <svg id="graph"></svg>
  </div>
</div>

<style>
  .container {
    text-align: center;
    margin-top: 50px;
  }

  .chart-title {
    margin-bottom: 20px;
  }

  .slider-container {
    margin-bottom: 20px;
  }

  #slider {
    width: 80%;
    margin: 0 auto;
  }

  #graph-container {
    width: 80%;
    margin: 0 auto;
  }

  svg {
    border: 1px solid #ccc;
  }
</style>
