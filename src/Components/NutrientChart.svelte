<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
  
    export let nutrients = {};
    export let ingredients = [];
    export let colorScale;
  
    let chartContainer;
  
    const nutrientNames = {
      calories: 'Calories',
      protein: 'Protein',
      fats: 'Fats',
      carbohydrates: 'Carbs.',
      sugar: 'Sugar',
      fiber: 'Fiber',
      sodium: 'Sodium',
      calcium: 'Calcium',
      vitaminC: 'Vit. C',
      vitaminB12: 'Vit. B12',
      iron: 'Iron',
      saturatedFat: 'Sat. Fat',
      water: 'Water'
    };
  
    onMount(() => {
      drawChart();
    });
  
    $: drawChart();
  
    afterUpdate(() => {
      drawChart();
    });
  
    function drawChart() {
      if (!chartContainer || !Object.keys(nutrients).length) return;
  
      d3.select(chartContainer).selectAll('*').remove();
  
      const margin = { top: 40, right: 40, bottom: 60, left: 50 },
        width = 1000 - margin.left - margin.right,
        height = 700 - margin.top - margin.bottom;
  
      const svg = d3.select(chartContainer)
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .style('display', 'block')
        .style('margin', 'auto')
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);
  
      const x = d3.scaleBand()
        .range([0, width])
        .domain(Object.keys(nutrientNames))
        .padding(0.2);
  
      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x).tickFormat(d => nutrientNames[d]))
        .selectAll('text')
        .style('font-size', '16px');
  
      const y = d3.scaleLinear()
        .domain([0, 200])
        .range([height, 0]);
  
      svg.append('g')
        .call(d3.axisLeft(y).tickFormat(d => `${d}%`))
        .selectAll('text')
        .style('font-size', '16px');
  
      let accumulatedNutrients = {};
      Object.keys(nutrientNames).forEach(nutrient => {
        accumulatedNutrients[nutrient] = ingredients.map(ingredient => {
          return {
            name: ingredient.name,
            nutrient: nutrient,
            value: (ingredient.nutrients[nutrient] * (ingredient.quantity / 100)) / dailyValues[nutrient] * 100
          };
        });
      });
  
      Object.keys(accumulatedNutrients).forEach((key, i) => {
        let cumulativeValue = 0;
        accumulatedNutrients[key].forEach((data, index) => {
          cumulativeValue += data.value;
  
          if (cumulativeValue > 100) {
            console.log(`Nutrient ${key} exceeded 100%: ${cumulativeValue}%`);
          }
  
          const rect = svg.append('rect')
            .attr('x', x(key))
            .attr('width', x.bandwidth())
            .attr('y', y(cumulativeValue))
            .attr('height', y(0) - y(data.value))
            .attr('fill', colorScale(data.name))
            .attr('class', cumulativeValue > 100 ? 'flashing' : '')
            .style('opacity', 1)
            .on('mouseover', function () {
              const tooltipText = `${data.name}:\n` +
                `  - Quantity: ${ingredients[index].quantity}g\n` +
                `  - ${key}: ${data.value.toFixed(2)}g\n` +
                `  - ${key} (% Daily Value): ${((data.value / 100) * dailyValues[key]).toFixed(2)}%\n`;
  
              const tooltipLines = tooltipText.split('\n');
  
              tooltipLines.forEach((line, index) => {
                svg.append('text')
                  .attr('class', 'tooltip')
                  .attr('x', x(key) + x.bandwidth() / 2)
                  .attr('y', y(cumulativeValue) - 50 + (index * 15))
                  .attr('text-anchor', 'middle')
                  .attr('fill', 'black')
                  .text(line);
              });
            })
            .on('mouseout', function () {
              svg.selectAll('.tooltip').remove();
            });
  
          if (cumulativeValue > 100) {
            // Trigger flashing effect using JavaScript
            setInterval(() => {
              const currentFill = rect.attr('fill');
              rect.attr('fill', currentFill === 'red' ? colorScale(data.name) : 'red');
            }, 500);
          }
        });
      });
  
      const legend = d3.select(chartContainer)
        .append('div')
        .attr('class', 'legend')
        .style('top', '20px')
        .style('right', '20px')
        .style('background-color', '#f4f2f9')
        .style('border', '1px solid #ddd')
        .style('padding', '25px');
  
      legend.append('h4').text('Selected Ingredients');
      const ingredientList = legend.append('ul');
  
      ingredients.forEach(ingredient => {
        const listItem = ingredientList.append('li');
        listItem.style('color', colorScale(ingredient.name))
          .text(`${ingredient.name} (${ingredient.quantity}g)`);
        listItem.append('input')
          .attr('type', 'number')
          .attr('min', '0')
          .attr('value', ingredient.quantity)
          .on('input', (event) => adjustQuantity(ingredient, event.target.value));
        listItem.append('button')
          .text('−')
          .style('color', 'red')
          .style('background', 'none')
          .style('border', 'none')
          .style('cursor', 'pointer')
          .style('margin-left', '5px')
          .on('click', () => removeIngredient(ingredient));
      });
    }
  
    function adjustQuantity(ingredient, newQuantity) {
      const index = ingredients.findIndex(item => item.name === ingredient.name);
  
      if (index !== -1) {
        ingredients[index].quantity = newQuantity;
        drawChart();
      }
    }
  
    function removeIngredient(ingredientToRemove) {
      ingredients = ingredients.filter(ingredient => ingredient !== ingredientToRemove);
      drawChart();
    }
  
    const dailyValues = {
      calories: 2000,
      protein: 50,
      fats: 70,
      carbohydrates: 300,
      sugar: 36,
      fiber: 25,
      sodium: 2300,
      calcium: 1000,
      vitaminC: 90,
      vitaminB12: 2.4,
      iron: 18,
      saturatedFat: 20,
      water: 3700
    };
  </script>
  
  <style>
    .chart-container {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      margin: auto;
      padding: 20px;
      width: 100%;
      max-width: 1200px;
      background-color: #f5f5f5;
    }
  
    .bar:hover {
      fill: #4e8fd3;
    }
  
    .legend {
      font-size: 14px;
    }
  
    .legend ul {
      list-style-type: none;
      padding: 0;
    }
  
    .legend li {
      display: flex;
      align-items: center;
    }
  
    .legend input {
      margin-left: 10px;
    }
  
    .legend button {
      margin-left: 5px;
      cursor: pointer;
    }
  </style>
  
  <div class="chart-container" bind:this={chartContainer}></div>
