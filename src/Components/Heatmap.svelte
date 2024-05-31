<style>
  /* Define styles for the tooltip */
  #tooltip {
    position: absolute;
    text-align: center;
    width: auto;
    height: auto;
    padding: 5px;
    background: lightsteelblue;
    border: 1px solid #ccc;
    border-radius: 5px;
    pointer-events: none;
    font-size: 12px;
  }

  /* Adjust font size and style for axis labels */
  .axis text {
    font-size: 12px;
    font-weight: bold;
  }

  /* Adjust font size and style for axis ticks */
  .axis path,
  .axis line {
    stroke: #ccc;
    fill: none;
    shape-rendering: crispEdges;
  }

  /* Container for heatmap and bar chart */
  .chart-container {
    display: flex;
    justify-content: space-between;
  }

  /* Adjust the size and margins of each chart */
  #heatmap, #bar-chart {
    width: 50%;
    height: 600px;
  }

  .bar {
    fill: steelblue;
  }
</style>

<div class="chart-container">
  <div id="heatmap"></div>
  <div id="bar-chart"></div>
</div>
<div id="tooltip"></div>

<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let data = [];
  let categories = ['BEEF', 'PORK', 'CEREALS RTE', 'BABYFOOD', 'LAMB', 'SOUP', 'CHICKEN', 'CEREALS', 'FAST FOODS', 'CANDIES', 'TURKEY', 'CAMPBELL SOUP COMPANY', 'BEANS', 'VEAL', "MCDONALD'S", 'CHEESE', 'POTATOES', 'COOKIES', 'ALCOHOLIC BEV', 'OIL'];
  let nutrients = ['Data.Protein', 'Data.Carbohydrate', 'Data.Fat.Total Lipid', 'Data.Sugar Total', 'Data.Fiber', 'Data.Major Minerals.Sodium', 'Data.Major Minerals.Calcium', 'Data.Vitamins.Vitamin C', 'Data.Vitamins.Vitamin B12', 'Data.Major Minerals.Iron', 'Data.Fat.Saturated Fat', 'Data.Water'];

  async function loadData() {
  try {
    data = await d3.csv('/dist/food.csv'); // Updated the path to your CSV file
    console.log('Data loaded successfully:', data);
    // Filter out data entries not belonging to specified categories
    data = data.filter(d => categories.includes(d.Category));
    drawHeatmap();
  } catch (error) {
    console.error('Error loading data:', error);
  }
}


  function drawHeatmap() {
    const margin = { top: 30, right: 30, bottom: 60, left: 60 };
    const width = 800 - margin.left - margin.right;
    const height = 600 - margin.top - margin.bottom;

    // Remove any existing SVG
    d3.select('#heatmap').selectAll('*').remove();

    const svg = d3.select('#heatmap')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    const x = d3.scaleBand()
      .domain(categories)
      .range([0, width])
      .padding(0.05);

    svg.append('g')
      .attr('transform', `translate(0, ${height})`)
      .call(d3.axisBottom(x))
      .selectAll('text')
      .attr('transform', 'translate(-10,0)rotate(-45)')
      .style('text-anchor', 'end');

    const y = d3.scaleBand()
      .domain(nutrients)
      .range([height, 0])
      .padding(0.05);

    svg.append('g')
      .call(d3.axisLeft(y));

    const nutrientAverages = {};
    nutrients.forEach(nutrient => {
      const values = data.map(d => +d[nutrient]);
      nutrientAverages[nutrient] = d3.mean(values.filter(v => !isNaN(v)));
    });

    const colorScales = {};
    nutrients.forEach(nutrient => {
      const nutrientAvg = nutrientAverages[nutrient];

      const colorScale = d3.scaleLinear()
          .domain([d3.min(data, d => +d[nutrient]), nutrientAvg, d3.max(data, d => +d[nutrient])])
          .range(['#f7fbff', '#08519c', '#001957']);

      colorScales[nutrient] = colorScale;
    });

    svg.selectAll()
      .data(data)
      .enter()
      .append('g')
      .selectAll('rect')
      .data(d => nutrients.map(n => ({ category: d.Category, nutrient: n, value: +d[n] })))
      .enter()
      .append('rect')
      .attr('x', d => x(d.category))
      .attr('y', d => y(d.nutrient))
      .attr('width', x.bandwidth())
      .attr('height', y.bandwidth())
      .style('fill', d => colorScales[d.nutrient](d.value))
      .style('stroke-width', 1)
      .style('stroke', 'white')
      .on('mouseover', (event, d) => {
        // Show tooltip with nutrient value and average value
        const categoryAvgValue = d3.mean(data.filter(item => item.Category === d.category).map(item => +item[d.nutrient]));
        const nutrientAvgValue = nutrientAverages[d.nutrient];
        const tooltip = d3.select('#tooltip');
        tooltip.style('opacity', 0.9)
          .html(`Category: ${d.category}<br>Nutrient: ${d.nutrient}<br>Category Average: ${categoryAvgValue.toFixed(2)}<br>Nutrient Average: ${nutrientAvgValue.toFixed(2)}`);
        tooltip.style('left', (event.pageX + 10) + 'px')
          .style('top', (event.pageY - 30) + 'px');
      })
      .on('mouseout', () => {
        // Hide tooltip on mouseout
        const tooltip = d3.select('#tooltip');
        tooltip.style('opacity', 0);
      })
      .on('click', (event, d) => {
          const category = d.category;
          const nutrient = d.nutrient;
          const topFoods = findTopFoodsInCategory(data, category, nutrient);
          displayBarChart(topFoods, nutrient, category);
      });
  }

  function findTopFoodsInCategory(data, category, nutrient) {
      // Filter data for the selected category
      const categoryData = data.filter(d => d.Category === category);

      // Sort the filtered data based on the selected nutrient
      const sortedData = categoryData.sort((a, b) => b[nutrient] - a[nutrient]);

      // Get the top 5 foods with the highest value for the selected nutrient
      return sortedData.slice(0, 5);
  }

  function displayBarChart(topFoods, nutrient, category) {
      drawBarChart(topFoods, nutrient, category);
  }

  function drawBarChart(topFoods, nutrient, category) {
      const margin = { top: 30, right: 30, bottom: 100, left: 60 }; // Increased bottom margin for x-axis labels
      const width = 800 - margin.left - margin.right;
      const height = 600 - margin.top - margin.bottom;

      // Remove any existing bar chart SVG
      d3.select('#bar-chart').selectAll('*').remove();

      const svg = d3.select('#bar-chart')
          .append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
          .append('g')
          .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = d3.scaleBand()
          .domain(topFoods.map(d => d.Description))
          .range([0, width])
          .padding(0.1);

      const y = d3.scaleLinear()
          .domain([0, d3.max(topFoods, d => parseFloat(d[nutrient]))])
          .nice()
          .range([height, 0]);

      svg.append('g')
          .attr('class', 'x-axis')
          .attr('transform', `translate(0, ${height})`)
          .call(d3.axisBottom(x))
          .selectAll('.x-axis text')
          .attr('transform', 'translate(-10,0)rotate(-45)')
          .style('text-anchor', 'end');

      svg.append('g')
          .attr('class', 'y-axis')
          .call(d3.axisLeft(y));

      svg.selectAll('.bar')
          .data(topFoods)
          .enter()
          .append('rect')
          .attr('class', 'bar')
          .attr('x', d => x(d.Description))
          .attr('y', d => y(parseFloat(d[nutrient])))
          .attr('width', x.bandwidth())
          .attr('height', d => height - y(parseFloat(d[nutrient])))
          .style('fill', 'steelblue');

// Calculate the average value for the nutrient in the selected category
const categoryAvgValue = d3.mean(topFoods.map(d => +d[nutrient]));

// Add the red horizontal line representing the average nutrient content
svg.append('line')
    .attr('x1', 0)
    .attr('x2', width)
    .attr('y1', y(categoryAvgValue))
    .attr('y2', y(categoryAvgValue))
    .attr('stroke', 'red')
    .attr('stroke-width', 2);

// Add a label for the red horizontal line
svg.append('text')
    .attr('x', width - 5)
    .attr('y', y(categoryAvgValue) - 10)
    .attr('text-anchor', 'end')
    .attr('fill', 'red')
    .text(`Nutritional Content Avg. Per Category: ${categoryAvgValue.toFixed(2)}`);
}

onMount(loadData);
</script>

