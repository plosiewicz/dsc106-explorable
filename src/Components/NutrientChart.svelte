User
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
  
    let flashingIntervals = [];
  
    onMount(() => {
      drawChart();
    });
  
    $: drawChart();
  
    afterUpdate(() => {
      drawChart();
    });
  
    function drawChart() {
      if (!chartContainer || !Object.keys(nutrients).length) return;
  
      // Clear previous intervals
      flashingIntervals.forEach(interval => clearInterval(interval));
      flashingIntervals = [];
  
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
            value: (ingredient.nutrients[nutrient] / dailyValues[nutrient]) * 100
          };
        });
      });
  
      Object.keys(accumulatedNutrients).forEach((key, i) => {
        let cumulativeValue = 0;
  
        accumulatedNutrients[key].forEach((data, index) => {
          cumulativeValue += data.value;
  
          const rect = svg.append('rect')
            .attr('x', x(key))
            .attr('width', x.bandwidth())
            .attr('y', y(cumulativeValue))
            .attr('height', y(0) - y(data.value))
            .attr('fill', colorScale(data.name))
            .style('opacity', 1)
            .on('mouseover', function () {
              const tooltipText = `${data.name}:\n` +
                `  - Quantity: (${ingredients[index].householdWeightDescription}) x ${ingredients[index].quantity}\n` +
                `  - ${nutrientNames[key]}: ${data.value.toFixed(2)}%\n`;
  
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
            const interval = setInterval(() => {
              const currentFill = rect.attr('fill');
              rect.attr('fill', currentFill === 'red' ? colorScale(data.name) : 'red');
            }, 500);
            flashingIntervals.push(interval);
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
  
      ingredients.forEach((ingredient, index) => {
        const listItem = ingredientList.append('li');
        listItem.style('color', colorScale(ingredient.name))
          .text(`${ingredient.name} (${ingredient.householdWeightDescription}) x ${ingredient.quantity}`);
        listItem.append('input')
          .attr('type', 'number')
          .attr('min', '0')
          .attr('value', ingredient.quantity)
          .on('input', (event) => adjustQuantity(index, event.target.value));
        listItem.append('button')
          .text('−')
          .style('color', 'red')
          .style('background', 'none')
          .style('border', 'none')
          .style('cursor', 'pointer')
          .style('margin-left', '5px')
          .on('click', () => removeIngredient(index));
      });
    }
  
    function adjustQuantity(index, newQuantity) {
      const ingredient = ingredients[index];
      const oldQuantity = ingredient.quantity;
      const parsedQuantity = parseFloat(newQuantity);
  
      if (!isNaN(parsedQuantity) && parsedQuantity > 0) {
        ingredients[index].quantity = parsedQuantity;
  
        // Recalculate the nutrients for the updated quantity
        ingredients[index].nutrients.calories = (ingredients[index].nutrients.calories / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.protein = (ingredients[index].nutrients.protein / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.fats = (ingredients[index].nutrients.fats / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.carbohydrates = (ingredients[index].nutrients.carbohydrates / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.sugar = (ingredients[index].nutrients.sugar / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.fiber = (ingredients[index].nutrients.fiber / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.sodium = (ingredients[index].nutrients.sodium / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.calcium = (ingredients[index].nutrients.calcium / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.vitaminC = (ingredients[index].nutrients.vitaminC / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.vitaminB12 = (ingredients[index].nutrients.vitaminB12 / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.iron = (ingredients[index].nutrients.iron / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.saturatedFat = (ingredients[index].nutrients.saturatedFat / oldQuantity) * parsedQuantity;
        ingredients[index].nutrients.water = (ingredients[index].nutrients.water / oldQuantity) * parsedQuantity;
  
        drawChart();
      }
    }
  
    function removeIngredient(index) {
      ingredients.splice(index, 1);
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