<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let data = [];
  let categories = ['BEEF', 'PORK', 'CEREALS RTE', 'BABYFOOD', 'LAMB', 'SOUP', 'CHICKEN', 'CEREALS', 'FAST FOODS', 'CANDIES', 'TURKEY', 'CAMPBELL SOUP COMPANY', 'BEANS', 'VEAL', "MCDONALD'S", 'CHEESE', 'POTATOES', 'COOKIES', 'ALCOHOLIC BEV', 'OIL'];
  let nutrients = ['Data.Protein', 'Data.Carbohydrate', 'Data.Fat.Total Lipid', 'Data.Sugar Total', 'Data.Fiber', 'Data.Major Minerals.Sodium', 'Data.Major Minerals.Calcium', 'Data.Vitamins.Vitamin C', 'Data.Vitamins.Vitamin B12', 'Data.Major Minerals.Iron', 'Data.Fat.Saturated Fat', 'Data.Water'];
  let nutrientMapping = {
    'Data.Protein': 'Protein (in g)',
    'Data.Carbohydrate': 'Carbohydrate (in g)',
    'Data.Fat.Total Lipid': 'Total Fat (in g)',
    'Data.Sugar Total': 'Sugar (in g)',
    'Data.Fiber': 'Fiber (in g)',
    'Data.Major Minerals.Sodium': 'Sodium (in mg)',
    'Data.Major Minerals.Calcium': 'Calcium (in mg)',
    'Data.Vitamins.Vitamin C': 'Vitamin C (in mg)',
    'Data.Vitamins.Vitamin B12': 'Vitamin B12 (in µg)',
    'Data.Major Minerals.Iron': 'Iron (in mg)' ,
    'Data.Fat.Saturated Fat': 'Saturated Fat (in g)',
    'Data.Water': 'Water (in g)'
  };

  async function loadData() {
    try {
      data = await d3.csv('/dsc106-explorable/data/food.csv'); // Updated the path to your CSV file
      console.log('Data loaded successfully:', data);
      // Filter out data entries not belonging to specified categories
      data = data.filter(d => categories.includes(d.Category));
      drawHeatmap();
    } catch (error) {
      console.error('Error loading data:', error);
    }
  }

  function drawHeatmap() {
    const margin = { top: 80, right: 30, bottom: 60, left: 100 }; // Increased top margin and left margin
    const width = 800 - margin.left - margin.right;
    const height = 600 - margin.top - margin.bottom;

    // Remove any existing SVG
    d3.select('#heatmap').selectAll('*').remove();

    const svg = d3.select('#heatmap')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .style('display', 'block')
      .style('margin', 'auto')
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
      .call(d3.axisLeft(y).tickFormat(d => nutrientMapping[d]));

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
          .html(`Category: ${d.category}<br>Nutrient: ${nutrientMapping[d.nutrient]}<br>Category Average: ${categoryAvgValue.toFixed(2)}<br>Nutrient Average: ${nutrientAvgValue.toFixed(2)}`);
        tooltip.style('left', (event.pageX + 10) + 'px')
          .style('top', (event.pageY - 30) + 'px');
      })
      .on('mouseout', () => {
        // Hide tooltip on mouseout
        const tooltip = d3.select('#tooltip');
        tooltip.style('opacity', 0);
      })
      .on('click', (event, d) => {
        // Call displayBarChart function
        displayBarChart(data.filter(item => item.Category === d.category), d.nutrient, d.category);
      });
  }

  function displayBarChart(categoryData, nutrient, category) {
      drawBarChart(categoryData, nutrient, category);
  }

  function drawBarChart(categoryData, nutrient, category) {
      const margin = { top: 30, right: 30, bottom: 100, left: 60 }; // Increased bottom margin for x-axis labels
      const width = 800 - margin.left - margin.right;
      const height = 600 - margin.top - margin.bottom;

      // Remove any existing bar chart SVG
      d3.select('#bar-chart').selectAll('*').remove();

      // Add dot element
      const svg = d3.select('#bar-chart')
          .append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
          .style('display', 'block')
          .style('margin', 'auto')
          .append('g')
          .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = d3.scaleBand()
          .domain(categoryData.map((d, i) => i)) // Use index as the domain for x scale
          .range([0, width])
          .padding(0.1);

      const y = d3.scaleLinear()
          .domain([0, d3.max(categoryData, d => parseFloat(d[nutrient]))])
          .nice()
          .range([height, 0]);

      svg.append('g')
          .attr('class', 'x-axis')
          .attr('transform', `translate(0, ${height})`)
          .call(d3.axisBottom(x).tickFormat('')) // Remove x-axis labels
          .selectAll('.x-axis text')
          .attr('transform', 'translate(-10,0)rotate(-45)')
          .style('text-anchor', 'end');

      svg.append('g')
          .attr('class', 'y-axis')
          .call(d3.axisLeft(y));

      // Add y-axis label
      svg.append('text')
          .attr('class', 'y-axis-label')
          .attr('transform', 'rotate(-90)')
          .attr('y', 0 - margin.left)
          .attr('x', 0 - (height / 2))
          .attr('dy', '1em')
          .style('text-anchor', 'middle')
          .text(nutrientMapping[nutrient]);

      // Add horizontal line element
      const horizontalLine = svg.append('line')
          .attr('class', 'horizontal-line')
          .attr('x1', 0)
          .attr('x2', width)
          .attr('stroke', 'black')
          .attr('stroke-width', 1)
          .attr('opacity', 0);

      svg.selectAll('.bar')
          .data(categoryData)
          .enter()
          .append('rect')
          .attr('class', 'bar')
          .attr('x', (d, i) => x(i))
          .attr('y', d => y(parseFloat(d[nutrient])))
          .attr('width', x.bandwidth())
          .attr('height', d => height - y(parseFloat(d[nutrient])))
          .style('fill', 'steelblue');

      // Add a rect to capture mouse events
      svg.append('rect')
          .attr('width', width)
          .attr('height', height)
          .style('fill', 'none')
          .style('pointer-events', 'all')
          .on('mousemove', function(event) {
            const [mouseX, mouseY] = d3.pointer(event);
            
            // Find the index of the bar that the mouse is over
            const hoveredIndex = Math.floor(mouseX / x.step());
            const barX = x(hoveredIndex) + x.bandwidth() / 2;

            // Calculate the y-coordinate of the top of the bar
            const barTopY = y(parseFloat(categoryData[hoveredIndex][nutrient]));

            // Show line, dot, and tooltip if the mouse x-coordinate lines up with the bar
            if (mouseX >= x(hoveredIndex) && mouseX <= x(hoveredIndex) + x.bandwidth()) {
                horizontalLine.attr('y1', barTopY)
                    .attr('y2', barTopY)
                    .attr('opacity', 1);

                svg.select('.dot')
                    .attr('cx', barX)
                    .attr('cy', barTopY)
                    .attr('opacity', 1);

                const tooltip = d3.select('#tooltip');
                tooltip.style('opacity', 0.9)
                    .html(`Food: ${categoryData[hoveredIndex].Description}<br>Nutrient: ${categoryData[hoveredIndex][nutrient]}`);
                tooltip.style('left', (event.pageX + 10) + 'px')
                    .style('top', (event.pageY - 30) + 'px');
            } else {
                // Hide line, dot, and tooltip if not hovering over the bar
                horizontalLine.attr('opacity', 0);
                svg.select('.dot').attr('opacity', 0);
                const tooltip = d3.select('#tooltip');
                tooltip.style('opacity', 0);
            }
        });

      // Move dot element to be on top of bars
      svg.append('circle')
        .attr('class', 'dot')
        .attr('r', 5)
        .attr('fill', 'red')
        .attr('opacity', 0);
  }

  onMount(() => {
      loadData();
  });
</script>

<style>
  #tooltip {position: absolute;
      text-align: center;
      width: 120px;
      height: auto;
      padding: 5px;
      font: 12px sans-serif;
      background: lightsteelblue;
      border: 0px;
      border-radius: 8px;
      pointer-events: none;
  }
</style>

<div id="heatmap"></div>
<div id="bar-chart"></div>
<div id="tooltip" style="opacity: 0;"></div>
