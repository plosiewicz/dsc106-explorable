<script>
  import { onMount, createEventDispatcher } from "svelte";
  import * as d3 from "d3";
  import NutrientChart from "./NutrientChart.svelte";

  let foodData = [];
  let searchQuery = "";
  let searchResults = [];
  let selectedNutrients = {};
  let selectedIngredients = []; // Array to store selected ingredients
  let colorScale = d3.scaleOrdinal(d3.schemeCategory10); // Color scale for ingredients

  const dispatch = createEventDispatcher(); // Create dispatcher

  onMount(async () => {
    const response = await fetch("/dsc106-explorable/data/food.csv");
    const data = await response.text();
    foodData = d3.csvParse(data, (d) => ({
      name: d.Description,
      calories: +d["Data.Kilocalories"],
      protein: +d["Data.Protein"],
      fats: +d["Data.Fat.Total Lipid"],
      carbohydrates: +d["Data.Carbohydrate"],
      servingSize: 100, // Assuming the data is in 100g serving
    }));
  });

  // Handle the search functionality
  function handleSearch() {
    if (searchQuery == ""){
      searchResults = [];
    }
    else{
    searchResults = foodData.filter((food) =>
      food.name.toLowerCase().includes(searchQuery.toLowerCase())).slice(0, 5);
    }
  }

  // Select a food item to display its nutrients
  function selectFood(food) {
    // Ask for quantity
    const quantity = prompt(`Enter quantity (in grams) of ${food.name}:`);
    if (quantity === null || quantity.trim() === "") return; // If quantity is not provided or canceled, do nothing

    // Convert quantity to a number
    const parsedQuantity = parseFloat(quantity);
    if (isNaN(parsedQuantity) || parsedQuantity <= 0) {
      alert("Please enter a valid quantity.");
      return; // If quantity is not a valid number or less than or equal to zero, show an alert and do nothing
    }

    // Calculate nutrients considering the quantity
    selectedNutrients = {
      calories: food.calories * parsedQuantity / food.servingSize,
      protein: food.protein * parsedQuantity / food.servingSize,
      fats: food.fats * parsedQuantity / food.servingSize,
      carbohydrates: food.carbohydrates * parsedQuantity / food.servingSize,
    };

    // Add selected food with quantity
    selectedIngredients.push({ name: food.name, quantity: parsedQuantity });

    // Clear the search bar and results
    searchQuery = "";
    searchResults = [];

    // Dispatch event to notify NutrientChart of ingredient update
    dispatch("ingredientsUpdated");
  }
  function showDetailedInfo(result) {
    const infoId = result.name.split(" ").join("-") + "-info";
    document.getElementById(infoId).style.display = "block";
  }

  function hideDetailedInfo(result) {
    const infoId = result.name.split(" ").join("-") + "-info";
    document.getElementById(infoId).style.display = "none";
  }

  // Add event listener for input event on the search input field
  function addInputEventListener() {
    const searchInput = document.querySelector('input[type="text"]');
    searchInput.addEventListener('input', handleSearch);
  }

  // Call the function to add the event listener when the component is mounted
  onMount(addInputEventListener);

</script>

<div class="search-container">
  <input
    type="text"
    placeholder="Search for food..."
    bind:value={searchQuery}
    on:input={handleSearch}
  />
  <ul>
    {#each searchResults as result}
      <li on:click={() => selectFood(result)}
          on:mouseover={() => showDetailedInfo(result)}
          on:mouseout={() => hideDetailedInfo(result)}>
        <strong>{result.name}</strong><br />
        <span>Calories: {result.calories}, Protein: {result.protein}g, Fats: {result.fats}g, Carbohydrates: {result.carbohydrates}g</span>
        <div class="detailed-info" id={result.name.split(" ").join("-") + "-info"} style="display:none;">
          <p>Calories per {result.servingSize}g: {result.calories}</p>
          <p>Protein per {result.servingSize}g: {result.protein}g</p>
          <p>Fats per {result.servingSize}g: {result.fats}g</p>
          <p>Carbohydrates per {result.servingSize}g: {result.carbohydrates}g</p>
        </div>
      </li>
    {/each}
  </ul>
  {#if Object.keys(selectedNutrients).length > 0}
    <NutrientChart
      nutrients={selectedNutrients}
      ingredients={selectedIngredients}
      {colorScale}
      on:ingredientsUpdated={() => {}} 
      />
  {/if}
</div>

<style>
  .search-container {
    padding: 20px;
    max-width: 600px;
    margin: auto;
  }
  input[type="text"] {
    width: 100%;
    padding: 10px;
    margin-bottom: 20px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  ul {
    list-style: none;
    padding: 0;
  }
  li {
    padding: 10px;
    border-bottom: 1px solid #ddd;
    cursor: pointer;
  }
  .detailed-info {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-top: 5px;
  }
</style>
