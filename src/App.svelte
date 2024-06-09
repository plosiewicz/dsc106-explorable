<script>
  import Meta from "./Meta.svelte";
  import Title from "./Components/Title.svelte";
  import Intro from "./Components/Intro.svelte";
  import TextAndMathEquations from "./Components/TextAndMathEquations.svelte";
  import LineChart from "./Components/LineChart.svelte";
  import LineGraph from './Components/LineGraph.svelte';
  import ScrollCenter from "./Components/ScrollCenter.svelte";
  import Conclusion from "./Components/Conclusion.svelte";
  import Resources from "./Components/Resources.svelte";
  import ScrollSide from "./Components/ScrollSide.svelte";
  import FoodSearch from "./Components/FoodSearch.svelte";
  import Heatmap from './Components/Heatmap.svelte';
  import KeyNutrientsBubbles from "./Components/KeyNutrientsBubbles.svelte";
  import NutrientsPieChart from "./Components/NutrientsPieChart.svelte";
  import NutrientsIntro from "./Components/NutrientsIntro.svelte";
</script>

<main>
  <h1>Nutrient Impact Explorer</h1>
  <Meta />
  <Title />
  <Intro />
  <KeyNutrientsBubbles />
  <NutrientsIntro />
  <section class="scroll-center-section">
    <ScrollCenter />
  </section>
  <section class="nutrients-pie-chart-section">
    <NutrientsPieChart />
  </section>
  <Heatmap />
  <div class="component food-search">
    <FoodSearch />
  </div>
  <Resources />
</main>

<style>
  main {
    font-family: 'Roboto', sans-serif;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
    color: #333;
    background-color: #cdb6cd;
  }

  h1 {
    text-align: center;
    font-size: 3.5rem;
    margin-bottom: 20px;
    color: #7b2666;
  }

  h2 {
    font-size: 2rem;
    margin-top: 40px;
    color: #2E3A59;
  }

  p {
    margin: 20px 0;
    color: #555;
  }

  .container {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .component {
    flex: 1 1 48%;
    margin: 10px 0;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    padding: 20px;
    border-radius: 8px;
    background-color: #fff;
    transition: box-shadow 0.3s, transform 0.3s;
  }

  section {
    margin: 40px 0; /* Add margin to sections to prevent overlapping */
    padding: 20px;
    border-radius: 8px;
    background-color: #fff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .scroll-center-section {
    min-height: 600px; /* Ensure minimum height */
  }

  .nutrients-pie-chart-section {
    min-height: 600px; /* Ensure minimum height */
  }

  .scroll-center-section + .nutrients-pie-chart-section {
    margin-top: 60px; /* Add extra space between specific sections */
  }
</style>
