<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';

    export let nutrients = {};
    export let ingredients = [];
    export let colorScale;

    let chartContainer;

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
    const margin = { top: 40, right: 40, bottom: 60, left: 120 }, // Increased left margin for nutrient values
        width = 1000 - margin.left - margin.right,
        height = 700 - margin.top - margin.bottom;

    // Append the SVG object to the div
    const svg = d3.select(chartContainer)
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

    // X axis
    const x = d3.scaleBand()
        .range([0, width])
        .domain(Object.keys(nutrients))
        .padding(0.2);

    svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x))
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

    // Stacked Bars
    let yOffset = 0; // Offset for stacking bars
    ingredients.forEach(ingredient => {
        Object.entries(nutrients).forEach(([key, value]) => {
            const nutrientValue = value * (ingredient.quantity / 100); // Calculate nutrient value for the ingredient

            // Format tooltip text
            const tooltipText = `${ingredient.name}:\n` +
                `  - Quantity: ${ingredient.quantity}g\n` +
                `  - ${key}: ${nutrientValue.toFixed(2)}g\n` + // Append nutrient value
                `  - ${key} (% Daily Value): ${(nutrientValue / dailyValues[key] * 100).toFixed(2)}%\n`; // Append % Daily Value

            const tooltipLines = tooltipText.split('\n'); // Split tooltip text into lines

            // Append tooltip lines
            tooltipLines.forEach((line, index) => {
                svg.append('text')
                    .attr('class', 'tooltip')
                    .attr('x', x(key) + x.bandwidth() / 2)
                    .attr('y', y((nutrientValue / dailyValues[key]) * 100) - yOffset - 5 + (index * 15)) // Adjust y-coordinate for each line
                    .attr('text-anchor', 'middle')
                    .attr('fill', 'black')
                    .text(line);
            });

            // Create a rectangle for each nutrient
            svg.append('rect')
                .attr('x', x(key))
                .attr('width', x.bandwidth())
                .attr('y', y((nutrientValue / dailyValues[key]) * 100)) // Adjust y-coordinate based on percentage
                .attr('height', height - y((nutrientValue / dailyValues[key]) * 100))
                .attr('fill', colorScale(ingredient.name))
                .attr('class', 'bar')
                .on('mouseover', function () {
                    // Append tooltip
                    tooltipLines.forEach((line, index) => {
                        svg.append('text')
                            .attr('class', 'tooltip')
                            .attr('x', x(key) + x.bandwidth() / 2)
                            .attr('y', y((nutrientValue / dailyValues[key]) * 100) - yOffset - 50 + (index * 15)) // Adjust y-coordinate for each line
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
        yOffset += 5; // Increment yOffset for stacking effect
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

    ingredients.forEach(ingredient => {
        const listItem = ingredientList.append('li');
        listItem.style('color', colorScale(ingredient.name))
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
        carbohydrates: 300 // Daily value for carbohydrates in grams
        // You can add more nutrients with their corresponding daily values here
    };
</script>

<style>
    .chart-container {
        position: relative;
        max-width: 600px;
        margin: auto;
    }

    .bar:hover {
        fill: #4e8fd3;
    }

    .legend {
        font-size: 14px;
    }
</style>

<div class="chart-container" bind:this={chartContainer}></div>
