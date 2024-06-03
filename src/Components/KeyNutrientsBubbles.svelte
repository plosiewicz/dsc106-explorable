<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let data = [
    {
      name: "Protein",
      description:
        "Proteins are the building blocks of life, essential for building and repairing tissues, producing enzymes and hormones. They play a crucial role in muscle development, immune function, and cell signaling. Sources of protein include meat, poultry, fish, eggs, dairy products, legumes, nuts, and seeds.",
      value: 40,
      importance: 5,
    },
    {
      name: "Carbohydrates",
      description:
        "Carbohydrates are the body's primary source of energy, providing fuel for physical activity, brain function, and vital organ operation. They come in two main forms: simple and complex. Simple carbohydrates, found in sugars, fruits, and some vegetables, provide quick energy but can lead to blood sugar spikes. Complex carbohydrates, found in whole grains, legumes, and starchy vegetables, provide sustained energy and important nutrients like fiber, vitamins, and minerals.",
      value: 30,
      importance: 3,
    },
    {
      name: "Fats",
      description:
        "Fats are essential for overall health, serving as a concentrated source of energy and playing a key role in cell structure, hormone production, and nutrient absorption. There are different types of fats, including saturated fats, unsaturated fats, and trans fats. While some fats, like monounsaturated and polyunsaturated fats found in nuts, seeds, avocados, and oily fish, are beneficial for heart health, excessive consumption of saturated and trans fats, found in red meat, dairy products, and processed foods, can increase the risk of heart disease and other health issues.",
      value: 20,
      importance: 4,
    },
    {
      name: "Vitamins and Minerals",
      description:
        "Vitamins and minerals are micronutrients that are essential for various physiological functions in the body. They play critical roles in metabolism, immune function, bone health, vision, and many other processes. Vitamins are organic compounds that are required in small amounts and include both water-soluble (e.g., vitamin C, B vitamins) and fat-soluble (e.g., vitamins A, D, E, K) vitamins. Minerals, on the other hand, are inorganic elements that are necessary for various bodily functions, such as calcium for bone health, iron for oxygen transport, and potassium for nerve function. Sources of vitamins and minerals include fruits, vegetables, whole grains, dairy products, meat, fish, nuts, and seeds.",
      value: 10,
      importance: 4,
    },
    {
      name: "Fiber",
      description:
        "Dietary fiber, also known as roughage or bulk, is a type of carbohydrate that the body cannot digest or absorb. It passes relatively intact through your stomach, small intestine, colon, and out of your body. Fiber is crucial for maintaining bowel regularity, preventing constipation, and promoting digestive health. Additionally, it can help lower cholesterol levels, control blood sugar levels, and aid in weight management by promoting feelings of fullness. Good sources of fiber include fruits, vegetables, whole grains, legumes, nuts, and seeds.",
      value: 15,
      importance: 4,
    },
    {
      name: "Sugar",
      description:
        "Sugar is a type of simple carbohydrate that provides a quick source of energy for the body. It occurs naturally in foods like fruits, vegetables, and dairy products, as well as added to processed foods and beverages. While small amounts of sugar can be part of a balanced diet, excessive intake of added sugars, especially from sugary beverages, sweets, and desserts, can contribute to weight gain, tooth decay, and various chronic diseases like obesity, type 2 diabetes, and heart disease.",
      value: 10,
      importance: 2,
    },
    {
      name: "Calcium",
      description:
        "Calcium is a mineral that is essential for building and maintaining strong bones and teeth. It also plays a critical role in muscle function, nerve signaling, blood clotting, and maintaining normal heart rhythm. Adequate calcium intake is particularly important during childhood and adolescence when bone mass is rapidly accumulating, as well as during adulthood and aging to prevent bone loss and reduce the risk of osteoporosis. Good dietary sources of calcium include dairy products like milk, yogurt, and cheese, as well as fortified foods, leafy greens, tofu, and almonds.",
      value: 25,
      importance: 4,
    },
    {
      name: "Iron",
      description:
        "Iron is a vital mineral that is necessary for the formation of hemoglobin, a protein in red blood cells that carries oxygen from the lungs to the rest of the body. It also plays a critical role in energy production, immune function, and cognitive development. Iron deficiency can lead to anemia, characterized by fatigue, weakness, and impaired cognitive function. Good dietary sources of iron include red meat, poultry, fish, lentils, beans, fortified cereals, and dark leafy greens. Consuming iron-rich foods with vitamin C-rich foods can enhance iron absorption.",
      value: 18,
      importance: 4,
    },
    {
      name: "Vitamin C",
      description:
        "Vitamin C, also known as ascorbic acid, is a water-soluble vitamin that functions as an antioxidant, helping to protect cells from damage caused by free radicals. It is also essential for collagen synthesis, wound healing, and the proper functioning of the immune system. Vitamin C aids in the absorption of iron from plant-based foods and supports the health of skin, teeth, gums, and blood vessels. While citrus fruits like oranges, lemons, and grapefruits are well-known sources of vitamin C, it can also be found in other fruits and vegetables such as strawberries, kiwi, bell peppers, broccoli, and tomatoes.",
      value: 12,
      importance: 4,
    },
    {
      name: "Vitamin A",
      description:
        "Vitamin A is a fat-soluble vitamin that is essential for vision, immune function, reproduction, and cellular communication. It plays a critical role in maintaining healthy skin and mucous membranes, promoting good vision, particularly in low-light conditions, and supporting immune responses to infections. Vitamin A can be obtained from two main sources: preformed vitamin A, found in animal-based foods like liver, eggs, and dairy products, and provitamin A carotenoids, found in colorful fruits and vegetables like carrots, sweet potatoes, spinach, and mangoes. Adequate vitamin A intake is crucial for overall health and wellbeing.",
      value: 8,
      importance: 4,
    },
    {
      name: "Vitamin D",
      description:
        'Vitamin D is a fat-soluble vitamin that plays a crucial role in calcium absorption, bone health, immune function, and cell growth regulation. It is often referred to as the "sunshine vitamin" because the body can produce it in response to sunlight exposure. In addition to sunlight, vitamin D can be obtained from dietary sources such as fatty fish (e.g., salmon, mackerel, tuna), fortified foods (e.g., milk, cereal, orange juice), and dietary supplements. Adequate vitamin D intake is essential for maintaining strong bones, preventing rickets in children and osteomalacia in adults, and reducing the risk of various chronic diseases, including osteoporosis, heart disease, diabetes, and certain cancers.',
      value: 7,
      importance: 3,
    },
  ];

  let selectedNutrient = null;

  onMount(() => {
    drawBubbleChart();
  });

  function drawBubbleChart() {
    const width = 1000;
    const height = 500;
    const margin = { top: 20, right: 20, bottom: 50, left: 20 };

    const colorScale = d3
      .scaleOrdinal()
      .domain([1, 2, 3, 4, 5])
      .range(["#FFCCCB", "#FF9999", "#FF6666", "#FF3333", "#FF0000"]);

    // Clear any existing SVG
    d3.select("#bubble-chart").selectAll("*").remove();

    // Set up the SVG container
    const svg = d3
      .select("#bubble-chart")
      .append("svg")
      .attr("width", width)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    const radiusScale = d3
      .scaleSqrt()
      .domain([0, d3.max(data, (d) => d.value)])
      .range([10, 50]);

    const simulation = d3
      .forceSimulation(data)
      .force(
        "x",
        d3
          .forceX((d) =>
            selectedNutrient && d.name === selectedNutrient.name
              ? 100
              : width / 2,
          )
          .strength(0.05),
      )
      .force("y", d3.forceY(height / 2).strength(0.05))
      .force(
        "collide",
        d3.forceCollide((d) => radiusScale(d.value) + 2),
      )
      .on("tick", ticked);

    const bubblesGroup = svg
      .selectAll(".bubble")
      .data(data)
      .enter()
      .append("g")
      .attr("class", "bubble-group")
      .on("mouseover", (event, d) => {
        d3.select(event.currentTarget).select("circle").style("opacity", 1);
      })
      .on("mouseout", (event, d) => {
        d3.select(event.currentTarget).select("circle").style("opacity", 0.7);
      })
      .on('click', (event, d) => {
        bubblesGroup.selectAll('circle').style('fill', d => colorScale(d.importance));

        selectedNutrient = d;
        simulation.alpha(1).restart();
        // Change color of selected bubble
        d3.select(event.currentTarget).select('circle').style('fill', '#8f7c75');
      });

      bubblesGroup.append('circle')
      .attr('class', d => selectedNutrient && d.name === selectedNutrient.name ? 'bubble selected-bubble' : 'bubble') // Updated class based on selection
      .attr('r', d => radiusScale(d.value))
      .style('fill', d => colorScale(d.importance))
      .style('opacity', 0.7);

    bubblesGroup
      .append("text")
      .attr("class", "bubble-label")
      .attr("text-anchor", "middle")
      .attr("dy", ".35em")
      .style("fill", "white")
      .text((d) => d.name);

    function ticked() {
      bubblesGroup.attr("transform", (d) => `translate(${d.x},${d.y})`);
    }
  }
</script>

<section>
  <h2><strong>Let's Explore Some Key Nutrients:</strong></h2>
  <div id="bubble-chart"></div>
</section>

{#if selectedNutrient}
  <section class="description-box">
    <h3><strong>{selectedNutrient.name}</strong></h3>
    <p>{selectedNutrient.description}</p>
  </section>
{/if}

<style>
  .bubble {
    cursor: pointer;
    transition:
      fill 0.3s,
      r 0.3s;
  }

  .bubble-label {
    font-size: 14px;
    pointer-events: none;
  }

  .bubble-group:hover .bubble {
    fill: #ff8c61;
  }

  .selected-bubble {
    fill: #8f7c75; /* Change to your desired color */
  }

  .description-box {
    background-color: #abbbb1;
    border: 1px solid #ccc;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    margin: 20px 0;
    padding: 20px;
    max-width: 500px;
    animation: fadeIn 0.5s ease-in-out;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .description-box h3 {
    margin-top: 0;
  }

  .description-box p {
    margin-bottom: 0;
  }
</style>
