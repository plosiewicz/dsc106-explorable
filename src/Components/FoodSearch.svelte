<script>
  import { onMount, createEventDispatcher } from "svelte";
  import { writable } from 'svelte/store';
  import * as d3 from "d3"; // Import d3 library
  import NutrientChart from "./NutrientChart.svelte";

  let foodData = [];
  let searchQuery = "";
  let searchResults = [];
  let currentPage = 0; // Current page of search results
  const resultsPerPage = 5; // Number of results to show per page

  let totalNutrients = {
    calories: 0,
    protein: 0,
    fats: 0,
    carbohydrates: 0,
    sugar: 0,
    fiber: 0,
    sodium: 0,
    calcium: 0,
    vitaminC: 0,
    vitaminB12: 0,
    iron: 0,
    saturatedFat: 0,
    water: 0
  };

  let selectedIngredients = writable([]); // Initialize as writable store
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
      sugar: +d["Data.Sugar Total"],
      fiber: +d["Data.Fiber"],
      sodium: +d["Data.Major Minerals.Sodium"],
      calcium: +d["Data.Major Minerals.Calcium"],
      vitaminC: +d["Data.Vitamins.Vitamin C"],
      vitaminB12: +d["Data.Vitamins.Vitamin B12"],
      iron: +d["Data.Major Minerals.Iron"],
      saturatedFat: +d["Data.Fat.Saturated Fat"],
      water: +d["Data.Water"],
      servingSize: 100, // Assuming the data is in 100g serving
    }));
  });

  // Handle the search functionality
  function handleSearch() {
    if (searchQuery.trim() === "") {
      searchResults = [];
      currentPage = 0;
    } else {
      searchResults = foodData.filter((food) =>
        food.name.toLowerCase().includes(searchQuery.toLowerCase())
      );
      currentPage = 0; // Reset to first page on new search
    }
  }

  // Navigate to the previous page
  function previousPage() {
    if (currentPage > 0) {
      currentPage--;
    }
  }

  // Navigate to the next page
  function nextPage() {
    if (currentPage < Math.ceil(searchResults.length / resultsPerPage) - 1) {
      currentPage++;
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
    const nutrients = {
      calories: food.calories * parsedQuantity / food.servingSize,
      protein: food.protein * parsedQuantity / food.servingSize,
      fats: food.fats * parsedQuantity / food.servingSize,
      carbohydrates: food.carbohydrates * parsedQuantity / food.servingSize,
      sugar: food.sugar * parsedQuantity / food.servingSize,
      fiber: food.fiber * parsedQuantity / food.servingSize,
      sodium: food.sodium * parsedQuantity / food.servingSize,
      calcium: food.calcium * parsedQuantity / food.servingSize,
      vitaminC: food.vitaminC * parsedQuantity / food.servingSize,
      vitaminB12: food.vitaminB12 * parsedQuantity / food.servingSize,
      iron: food.iron * parsedQuantity / food.servingSize,
      saturatedFat: food.saturatedFat * parsedQuantity / food.servingSize,
      water: food.water * parsedQuantity / food.servingSize,
    };

    // Update total nutrients
    totalNutrients.calories += nutrients.calories;
    totalNutrients.protein += nutrients.protein;
    totalNutrients.fats += nutrients.fats;
    totalNutrients.carbohydrates += nutrients.carbohydrates;
    totalNutrients.sugar += nutrients.sugar;
    totalNutrients.fiber += nutrients.fiber;
    totalNutrients.sodium += nutrients.sodium;
    totalNutrients.calcium += nutrients.calcium;
    totalNutrients.vitaminC += nutrients.vitaminC;
    totalNutrients.vitaminB12 += nutrients.vitaminB12;
    totalNutrients.iron += nutrients.iron;
    totalNutrients.saturatedFat += nutrients.saturatedFat;
    totalNutrients.water += nutrients.water;

    // Add selected food with quantity and nutrients
    selectedIngredients.update(arr => [...arr, { name: food.name, quantity: parsedQuantity, nutrients }]);

    // Clear the search bar and results
    searchQuery = "";
    searchResults = [];

    // Dispatch event to notify NutrientChart of ingredient update
    dispatch("ingredientsUpdated");
  }
</script>

<div class="search-container">
  <input
    type="text"
    placeholder="Search for food..."
    bind:value={searchQuery}
    on:input={handleSearch}
  />
  <ul>
    {#each searchResults.slice(currentPage * resultsPerPage, (currentPage + 1) * resultsPerPage) as result}
      <li on:click={() => selectFood(result)}>
        <strong>{result.name}</strong><br />
        <span>Calories: {result.calories}, Protein: {result.protein}g, Fats: {result.fats}g, Carbohydrates: {result.carbohydrates}g</span>
      </li>
    {/each}
  </ul>
  <div class="pagination">
    {#if currentPage > 0}
      <button on:click={previousPage}>Previous</button>
    {/if}
    {#if currentPage < Math.ceil(searchResults.length / resultsPerPage) - 1}
      <button on:click={nextPage}>Next</button>
    {/if}
  </div>
  {#if $selectedIngredients.length > 0}
    <NutrientChart
      nutrients={totalNutrients}
      ingredients={$selectedIngredients}
      colorScale={colorScale}
      on:ingredientsUpdated={() => {}}
    />
  {/if}
</div>

<style>
  .search-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    width: 100%;
    max-width: 1000px; /* Ensure enough width */
    background-color: #f5f5f5; /* Set background to light gray */
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
    width: 100%;
  }
  li {
    padding: 10px;
    border-bottom: 1px solid #ddd;
    cursor: pointer;
    width: 100%;
    text-align: left;
  }
  .pagination {
    margin-top: 10px;
    display: flex;
    justify-content: space-between;
    width: 100%;
  }
  .pagination button {
    padding: 5px 10px;
    font-size: 14px;
  }
</style>


