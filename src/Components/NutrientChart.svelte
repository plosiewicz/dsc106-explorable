<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
  
    export let nutrients = {};
    export let ingredients = [];
    export let colorScale;
  
    let chartContainer;
    let infoText = ''; // This will store the information to display
  
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
            .on('click', function () {
              if (cumulativeValue >= 75 && cumulativeValue <= 125) {
                switch (key) {
                  case 'calories':
                    infoText = `Effects of healthy Calories levels:\n• Provides energy\n• Supports bodily functions\n• Maintains weight`;
                    break;
                  case 'protein':
                    infoText = `Effects of healthy Protein levels:\n• Builds and repairs tissues\n• Supports immune function\n• Maintains muscle mass`;
                    break;
                  case 'fats':
                    infoText = `Effects of healthy Fats levels:\n• Provides energy\n• Supports cell growth\n• Protects organs`;
                    break;
                  case 'carbohydrates':
                    infoText = `Effects of healthy Carbohydrates levels:\n• Provides energy\n• Supports brain function\n• Maintains physical activity`;
                    break;
                  case 'sugar':
                    infoText = `Effects of healthy Sugar levels:\n• Provides quick energy\n• Enhances mood\n• Supports brain function`;
                    break;
                  case 'fiber':
                    infoText = `Effects of healthy Fiber levels:\n• Aids digestion\n• Regulates blood sugar\n• Lowers cholesterol`;
                    break;
                  case 'sodium':
                    infoText = `Effects of healthy Sodium levels:\n• Balances fluids\n• Supports nerve function\n• Maintains muscle function`;
                    break;
                  case 'calcium':
                    infoText = `Effects of healthy Calcium levels:\n• Strengthens bones\n• Supports muscle function\n• Aids nerve signaling`;
                    break;
                  case 'vitaminC':
                    infoText = `Effects of healthy Vitamin C levels:\n• Boosts immunity\n• Aids iron absorption\n• Promotes skin health`;
                    break;
                  case 'vitaminB12':
                    infoText = `Effects of healthy Vitamin B12 levels:\n• Supports nerve function\n• Aids red blood cell formation\n• Boosts energy`;
                    break;
                  case 'iron':
                    infoText = `Effects of healthy Iron levels:\n• Prevents anemia\n• Supports oxygen transport\n• Boosts energy`;
                    break;
                  case 'saturatedFat':
                    infoText = `Effects of healthy Saturated Fat levels:\n• Provides energy\n• Supports cell function\n• Maintains hormone levels`;
                    break;
                  case 'water':
                    infoText = `Effects of healthy Water levels:\n• Maintains hydration\n• Supports bodily functions\n• Regulates temperature`;
                    break;
                  default:
                    infoText = '';
                }
              } else if (cumulativeValue < 75) {
                switch (key) {
                  case 'calories':
                    infoText = `Effects of Calorie deficiency:\n• Fatigue\n• Weight loss\n• Weakened immune system`;
                    break;
                  case 'protein':
                    infoText = `Effects of Protein deficiency:\n• Muscle loss\n• Weak immune system\n• Edema`;
                    break;
                  case 'fats':
                    infoText = `Effects of Fat deficiency:\n• Dry skin\n• Hair loss\n• Hormonal imbalances`;
                    break;
                  case 'carbohydrates':
                    infoText = `Effects of Carbohydrate deficiency:\n• Fatigue\n• Mental fog\n• Ketosis`;
                    break;
                  case 'sugar':
                    infoText = `Effects of Sugar deficiency:\n• Low energy\n• Mental fatigue\n• Hypoglycemia`;
                    break;
                  case 'fiber':
                    infoText = `Effects of Fiber deficiency:\n• Constipation\n• Blood sugar spikes\n• Increased cholesterol`;
                    break;
                  case 'sodium':
                    infoText = `Effects of Sodium deficiency:\n• Hyponatremia\n• Muscle cramps\n• Fatigue`;
                    break;
                  case 'calcium':
                    infoText = `Effects of Calcium deficiency:\n• Weak bones\n• Muscle cramps\n• Osteoporosis`;
                    break;
                  case 'vitaminC':
                    infoText = `Effects of Vitamin C deficiency:\n• Scurvy\n• Weak immunity\n• Slow wound healing`;
                    break;
                  case 'vitaminB12':
                    infoText = `Effects of Vitamin B12 deficiency:\n• Anemia\n• Fatigue\n• Nerve damage`;
                    break;
                  case 'iron':
                    infoText = `Effects of Iron deficiency:\n• Anemia\n• Fatigue\n• Weak immunity`;
                    break;
                  case 'saturatedFat':
                    infoText = `Effects of Saturated Fat deficiency:\n• Lack of energy\n• Hormonal imbalances\n• Vitamin deficiencies`;
                    break;
                  case 'water':
                    infoText = `Effects of Water deficiency:\n• Dehydration\n• Kidney stones\n• Constipation`;
                    break;
                  default:
                    infoText = '';
                }
              } else {
                switch (key) {
                  case 'calories':
                    infoText = `Effects of excessive Calories:\n• Weight gain\n• Obesity\n• Increased disease risk`;
                    break;
                  case 'protein':
                    infoText = `Effects of excessive Protein:\n• Kidney strain\n• Dehydration\n• Digestive issues`;
                    break;
                  case 'fats':
                    infoText = `Effects of excessive Fats:\n• Weight gain\n• Heart disease\n• High cholesterol`;
                    break;
                  case 'carbohydrates':
                    infoText = `Effects of excessive Carbohydrates:\n• Weight gain\n• Insulin resistance\n• Blood sugar spikes`;
                    break;
                  case 'sugar':
                    infoText = `Effects of excessive Sugar:\n• Weight gain\n• Diabetes\n• Tooth decay`;
                    break;
                  case 'fiber':
                    infoText = `Effects of excessive Fiber:\n• Digestive discomfort\n• Nutrient absorption issues\n• Bloating`;
                    break;
                  case 'sodium':
                    infoText = `Effects of excessive Sodium:\n• High blood pressure\n• Heart disease\n• Stroke`;
                    break;
                    case 'calcium':
                    infoText = `Effects of excessive Calcium:\n• Kidney stones\n• Heart issues\n• Impaired absorption of other minerals`;
                    break;
                  case 'vitaminC':
                    infoText = `Effects of excessive Vitamin C:\n• Diarrhea\n• Nausea\n• Stomach cramps`;
                    break;
                  case 'vitaminB12':
                    infoText = `Effects of excessive Vitamin B12:\n• Acne\n• Rosacea\n• Nerve issues`;
                    break;
                  case 'iron':
                    infoText = `Effects of excessive Iron:\n• Liver damage\n• Heart problems\n• Diabetes`;
                    break;
                  case 'saturatedFat':
                    infoText = `Effects of excessive Saturated Fat:\n• Heart disease\n• High cholesterol\n• Weight gain`;
                    break;
                  case 'water':
                    infoText = `Effects of excessive Water:\n• Hyponatremia\n• Swelling\n• Electrolyte imbalance`;
                    break;
                  default:
                    infoText = '';
                }
            }
            updateInfoText(infoText);
          })
          .on('mouseover', function (event) {
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

  function updateInfoText(text) {
    document.querySelector('.info-text').innerText = text;
  }
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

  .info-text {
    margin-top: 20px;
    font-size: 16px;
    background-color: #fff;
    padding: 10px;
    border: 1px solid #ddd;
  }
</style>

<div class="chart-container" bind:this={chartContainer}></div>
<div class="info-text"></div>
