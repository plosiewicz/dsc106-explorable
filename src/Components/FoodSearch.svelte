<script>
  import { onMount, onDestroy, createEventDispatcher } from "svelte";
  import * as d3 from "d3";
  import { writable } from 'svelte/store';
  import NutrientChart from "./NutrientChart.svelte";

  let foodData = [];
  let searchQuery = "";
  let searchResults = [];
  let totalNutrients = { calories: 0, protein: 0, fats: 0, carbohydrates: 0 };
  let selectedIngredients = writable([]); // Initialize as writable store
  let colorScale = writable(d3.scaleOrdinal(d3.schemeCategory10)); // Color scale for ingredients

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
    
    window.addEventListener("scroll", handleScroll);
  });

  onDestroy(() => {
    window.removeEventListener("scroll", handleScroll);
  });

  // Handle the search functionality
  function handleSearch() {
    if (searchQuery.trim() === "") {
      searchResults = [];
    } else {
      searchResults = foodData.filter((food) =>
        food.name.toLowerCase().includes(searchQuery.toLowerCase())
      ).slice(0, 5);
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
    };

    // Update total nutrients
    totalNutrients.calories += nutrients.calories;
    totalNutrients.protein += nutrients.protein;
    totalNutrients.fats += nutrients.fats;
    totalNutrients.carbohydrates += nutrients.carbohydrates;

    // Add selected food with quantity and nutrients
    selectedIngredients.update(arr => [...arr, { name: food.name, quantity: parsedQuantity, nutrients }]);

    // Clear the search bar and results
    searchQuery = "";
    searchResults = [];

    // Dispatch event to notify NutrientChart of ingredient update
    dispatch("ingredientsUpdated");
  }

  // Handle scrolling
  function handleScroll() {
    if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) {
      // At the bottom of the page
      console.log("Bottom of the page reached");
    }
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
    {#each searchResults as result}
      <li on:click={() => selectFood(result)}>
        <strong>{result.name}</strong><br />
        <span>Calories: {result.calories}, Protein: {result.protein}g, Fats: {result.fats}g, Carbohydrates: {result.carbohydrates}g</span>
      </li>
    {/each}
  </ul>
  {#if $selectedIngredients.length > 0}
    <NutrientChart
      nutrients={totalNutrients}
      ingredients={$selectedIngredients}
      colorScale={$colorScale}
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
</style>
