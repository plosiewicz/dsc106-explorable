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
</script>

<main>
  <h1>Nutrient Impact Explorer</h1>
  <Meta />
  <Title />
  <Intro />
  <KeyNutrientsBubbles />
  <ScrollCenter />
  <Heatmap />
  <ScrollSide />
  <FoodSearch />
  <LineChart />
  <Conclusion />
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

  /* Flexbox layout for components */
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

  .component:hover {
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
    transform: translateY(-5px);
  }

  /* Button styles */
  button {
    background-color: #FF6F61;
    color: white;
    border: none;
    padding: 10px 20px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 16px;
    transition: background-color 0.3s;
  }

  button:hover {
    background-color: #E65C50;
  }

  /* Responsive design */
  @media (max-width: 768px) {
    .component {
      flex: 1 1 100%;
    }

    h1 {
      font-size: 2rem;
    }

    h2 {
      font-size: 1.5rem;
    }
  }

  /* Additional component-specific styles */
  /* ScrollCenter styles */
  .scroll-center {
    background-color: #FFF3E0;
    padding: 20px;
    border-radius: 8px;
  }

  /* Heatmap styles */
  .heatmap {
    background-color: #E3F2FD;
    padding: 20px;
    border-radius: 8px;
  }

  /* FoodSearch styles */
  .food-search {
    background-color: #316e9c;
    padding: 20px;
    border-radius: 8px;
  }

  /* LineChart styles */
  .line-chart {
    background-color: #FCE4EC;
    padding: 20px;
    border-radius: 8px;
  }

  /* Conclusion styles */
  .conclusion {
    background-color: #e7f5ff;
    padding: 20px;
    border-radius: 8px;
  }
</style>
