<script>
    import { onMount } from 'svelte';
    import { pie } from 'd3-shape';
    import { select } from 'd3-selection';
    import { arc } from 'd3-shape';
    import { format } from 'd3-format';
  
    let dietData = [];
    let selectedDiet = '';
  
    const colors = ['#1f77b4', '#ff7f0e', '#2ca02c']; // Colors for pie chart segments
  
    const diets = {
      'Cardiac Diet': { protein: 21.8, carbohydrates: 47.3, fats: 40.9 },
      'Keto Diet': { protein: 22.1, carbohydrates: 5.9, fats: 79.4 },
      'Paleo Diet': { protein: 24.1, carbohydrates: 12.0, fats: 67.7 },
      'Plant Based Diet': { protein: 16.7, carbohydrates: 36.4, fats: 30.7 }
    };
  
    const dietDescriptions = {
      'Cardiac Diet': `The cardiac diet, essential for heart health, prioritizes foods low in saturated fats, cholesterol, and sodium, while emphasizing the consumption of fruits, vegetables, whole grains, lean proteins (like poultry and fish), and healthy fats (found in nuts, seeds, and olive oil). It restricts processed foods, red meats, fried items, sugary beverages, and high-sodium foods, advocating portion control and whole, nutrient-rich foods to support cardiovascular well-being.`,
      'Keto Diet': `The ketogenic (keto) diet is a high-fat, low-carbohydrate eating plan designed to shift the body's metabolism into a state called ketosis, where fat becomes the primary source of energy. With a typical macronutrient breakdown of 70-80% fat, 10-20% protein, and 5-10% carbohydrates, the diet restricts carb intake to around 20-50 grams per day. This prompts the liver to produce ketones from fat, which fuel the body and brain instead of glucose derived from carbs. People follow the keto diet for weight loss, improved blood sugar control, enhanced mental clarity, and increased energy levels. However, it's essential to consult with a healthcare professional before starting, and to maintain proper hydration and electrolyte balance to prevent potential side effects.`,
      'Paleo Diet': `Rooted in the presumed dietary habits of Paleolithic-era humans, the Paleo diet centers on foods that could have been hunted, fished, or gathered, such as lean meats, fish, seafood, fruits, vegetables, nuts, and seeds. Conversely, it avoids or minimizes foods that emerged with agricultural practices, including grains, legumes, dairy products, refined sugars, and processed foods. While proponents argue that it aligns with human evolutionary patterns and can reduce inflammation, support weight loss, and improve metabolic function, critics caution about its historical accuracy and potential nutritional deficiencies if not carefully balanced.`,
      'Plant Based Diet': `A plant-based diet emphasizes plant-derived foods like fruits, vegetables, whole grains, legumes, nuts, and seeds, while minimizing or excluding animal products. Ranging from flexitarian to strict vegan, it's associated with numerous health benefits, including reduced risk of heart disease, certain cancers, and type 2 diabetes, as well as potential weight management advantages. However, it requires attention to ensure adequate intake of essential nutrients like protein, iron, calcium, vitamin B12, and omega-3 fatty acids, often necessitating careful planning and supplementation.`
    };
  
    const createPieChart = () => {
      const width = 300;
      const height = 300;
      const radius = Math.min(width, height) / 2;
  
      const svg = select('.pie-chart')
        .attr('width', width)
        .attr('height', height)
        .append('g')
        .attr('transform', `translate(${width / 2}, ${height / 2})`);
  
      const data = dietData.map(d => d.value);
  
      const pieGenerator = pie().value(d => d);
  
      const pieData = pieGenerator(data);
  
      const arcGenerator = arc()
        .innerRadius(0)
        .outerRadius(radius);
  
      const arcs = svg.selectAll('.arc')
        .data(pieData)
        .enter()
        .append('g')
        .attr('class', 'arc');
  
      arcs.append('path')
        .attr('d', arcGenerator)
        .attr('fill', (d, i) => colors[i])
        .on('mouseover', function(d) {
          const tooltip = select('.tooltip');
          const percentage = format('.1f')(100 * d.data / dietData.reduce((acc, curr) => acc + curr.value, 0));
          tooltip.html(`${percentage}%`);
          tooltip.style('display', 'block');
        })
        .on('mousemove', function() {
          const tooltip = select('.tooltip');
          tooltip.style('top', (event.pageY - 10) + 'px')
            .style('left', (event.pageX + 10) + 'px');
        })
        .on('mouseout', function() {
          select('.tooltip').style('display', 'none');
        });
  
      arcs.append('text')
        .attr('transform', d => `translate(${arcGenerator.centroid(d)})`)
        .attr('text-anchor', 'middle')
        .attr('font-size', '12px')
        .attr('fill', 'white')
        .text((d, i) => `${Object.keys(diets[selectedDiet])[i]}: ${dietData[i].value}%`);
    };
  
    const updatePieChart = () => {
      dietData = Object.keys(diets[selectedDiet]).map(key => ({ nutrient: key, value: diets[selectedDiet][key] }));
      select('.pie-chart').selectAll('*').remove();
      createPieChart();
    };
  
    const handleDietClick = (diet) => {
      selectedDiet = diet;
      updatePieChart();
    };
  
    onMount(() => {
      handleDietClick('Cardiac Diet'); // Default to Cardiac Diet
    });
</script>
  
<style>
    .container {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
  
    .pie-chart {
      margin-top: 20px;
    }
  
    .button-container {
      display: flex;
      justify-content: space-evenly;
      width: 100%;
      margin-top: 20px;
    }
  
    button {
      padding: 10px 20px;
      cursor: pointer;
      border: none;
      border-radius: 5px;
      background-color: #007bff;
      color: white;
      font-size: 16px;
      transition: background-color 0.3s ease;
    }
  
    button:hover {
      background-color: #0056b3;
    }
  
    .description-box {
      margin-top: 20px;
      width: 100%;
      display: flex;
      justify-content: center;
    }
  
    .white-box {
      background-color: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      max-width: 800px;
      margin: auto;
    }
  
    .description {
      text-align: center;
    }
  
    .tooltip {
      position: absolute;
      background-color: rgba(0, 0, 0, 0.7);
      color: white;
      padding: 5px;
      border-radius: 5px;
      display: none;
      pointer-events: none;
    }
</style>
  
<div class="container">
    <div class="button-container">
        {#each Object.keys(diets) as diet}
            <button class:selected={selectedDiet === diet} on:click={() => handleDietClick(diet)}>{diet}</button>
        {/each}
    </div>
    <svg class="pie-chart"></svg>
    {#if selectedDiet}
        <div class="description-box">
            <div class="white-box">
                <p class="description">{dietDescriptions[selectedDiet]}</p>
            </div>
        </div>
    {/if}
    <div class="tooltip"></div>
</div>
