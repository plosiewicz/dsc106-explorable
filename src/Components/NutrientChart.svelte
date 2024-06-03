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
  
    // Redraw the chart whenever the nutrients or ingredients change
    $: drawChart();
  
    // Redraw the chart after each update
    afterUpdate(() => {
      drawChart();
    });
  
    // Function to adjust quantity of an ingredient
    function adjustQuantity(ingredient, newQuantity) {
      // Find the index of the ingredient
      const index = ingredients.findIndex(item => item === ingredient);
  
      // Update the quantity of the ingredient
      if (index !== -1) {
        ingredients[index].quantity = newQuantity;
        // Redraw the chart
        drawChart();
      }
    }
  
    function drawChart() {
      if (!chartContainer || !Object.keys(nutrients).length) return;
  
      // Clear the previous chart
      d3.select(chartContainer).selectAll('*').remove();
  
      // Set the dimensions and margins of the chart
      const margin = { top: 40, right: 40, bottom: 60, left: 50 }, // Adjusted left margin for nutrient values
        width = 1000 - margin.left - margin.right,
        height = 700 - margin.top - margin.bottom;
  
      // Append the SVG object to the div
      const svg = d3.select(chartContainer)
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .style('display', 'block')
        .style('margin', 'auto')
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);
  
      // X axis
      const x = d3.scaleBand()
        .range([0, width])
        .domain(Object.keys(nutrients))
        .padding(0.2);
  
      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x).tickFormat(d => nutrientNames[d])) // Use nutrientNames mapping for x-axis labels
        .selectAll('text')
        .style('font-size', '16px');
  
      // Add Y axis
      const y = d3.scaleLinear()
        .domain([0, 100]) // Percentage scale
        .range([height, 0]);
  
      svg.append('g')
        .call(d3.axisLeft(y).tickFormat(d => `${d}%`)) // Format ticks as percentages
        .selectAll('text')
        .style('font-size', '16px');
  
      // Accumulate nutrient values for stacking
      let accumulatedNutrients = {};
      Object.keys(nutrients).forEach(nutrient => {
        accumulatedNutrients[nutrient] = ingredients.map(ingredient => {
          return {
            name: ingredient.name,
            nutrient: nutrient,
            value: (ingredient.nutrients[nutrient] * (ingredient.quantity / 100)) / dailyValues[nutrient] * 100 // Calculate percentage of daily value
          };
        });
      });
  
      // Stacked Bars
      Object.keys(accumulatedNutrients).forEach((key, i) => {
        let cumulativeValue = 0;
        accumulatedNutrients[key].forEach((data, index) => {
          cumulativeValue += data.value; // Accumulate the values
  
          svg.append('rect')
            .attr('x', x(key))
            .attr('width', x.bandwidth())
            .attr('y', y(cumulativeValue))
            .attr('height', y(0) - y(data.value))
            .attr('fill', colorScale(index)) // Use index to ensure different color for each food
            .attr('class', 'bar')
            .on('mouseover', function () {
              // Format tooltip text
              const tooltipText = `${data.name}:\n` +
                `  - Quantity: ${ingredients[index].quantity}g\n` +
                `  - ${nutrientNames[key]}: ${data.value.toFixed(2)}g\n` + // Append nutrient value with display name
                `  - ${nutrientNames[key]} (% Daily Value): ${((data.value / 100) * dailyValues[key]).toFixed(2)}%\n`; // Append % Daily Value
  
              const tooltipLines = tooltipText.split('\n'); // Split tooltip text into lines
  
              // Append tooltip lines
              tooltipLines.forEach((line, index) => {
                svg.append('text')
                  .attr('class', 'tooltip')
                  .attr('x', x(key) + x.bandwidth() / 2)
                  .attr('y', y(cumulativeValue) - 50 + (index * 15)) // Adjust y-coordinate for each line
                  .attr('text-anchor', 'middle')
                  .attr('fill', 'black')
                  .text(line);
              });
            })
            .on('mouseout', function () {
              // Remove tooltip
              svg.selectAll('.tooltip').remove();
            });
        });
      });
  
      // Legend
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
        listItem.style('color', colorScale(index)) // Use index to ensure different color for each food
          .text(`${ingredient.name} (${ingredient.quantity}g)`); // Display quantity
        // Add input field to adjust quantity
        listItem.append('input')
          .attr('type', 'number')
          .attr('min', '0')
          .attr('value', ingredient.quantity)
          .on('input', (event) => adjustQuantity(ingredient, event.target.value));
        // Add remove button as a minus sign
        listItem.append('button')
          .text('−')
          .style('color', 'red') // Style the minus sign
          .style('background', 'none')
          .style('border', 'none')
          .style('cursor', 'pointer')
          .style('margin-left', '5px') // Add some margin
          .on('click', () => removeIngredient(ingredient));
      });
    }
  
    // Function to remove ingredient
    function removeIngredient(ingredientToRemove) {
      ingredients = ingredients.filter(ingredient => ingredient !== ingredientToRemove);
    }
  
    // Object containing daily values for each nutrient (adjust values as needed)
    const dailyValues = {
      calories: 2000,
      protein: 50, // Daily value for protein in grams
      fats: 70, // Daily value for fats in grams
      carbohydrates: 300, // Daily value for carbohydrates in grams
      sugar: 50, // Daily value for sugar in grams
      fiber: 25, // Daily value for fiber in grams
      sodium: 2300, // Daily value for sodium in milligrams
      calcium: 1000, // Daily value for calcium in milligrams
      vitaminC: 90, // Daily value for vitamin C in milligrams
      vitaminB12: 2.4, // Daily value for vitamin B12 in micrograms
      iron: 18, // Daily value for iron in milligrams
      saturatedFat: 20, // Daily value for saturated fat in grams
      water: 3700 // Daily value for water in grams
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
      max-width: 1200px; /* Added to control the width */
      background-color: #f5f5f5; /* Set background to light gray */
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
  