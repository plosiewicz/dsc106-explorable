<script>
  import Scrolly from "./Scrolly.svelte";
  import { select } from "d3-selection";

  // Paragraph text for scrolly
  $: steps = [
    `<h1 class='step-title'>Macronutrients</h1>
    <br><br>
    <p>
      "Macronutrients are the essential nutrients that the body requires in significant amounts 
        to generate energy and sustain key physiological functions. These include carbohydrates, proteins, 
        and fats. Carbohydrates, present in foods like grains, fruits, and vegetables, serve as the body's 
        primary energy source. Proteins, composed of amino acids, are critical for tissue growth, repair, 
        and maintenance, and can be found in meats, dairy, legumes, and nuts. Fats, though often misunderstood, 
        are vital for vitamin absorption, organ protection, and long-term energy storage, and are sourced 
        from oils, butter, avocados, and fatty fish. A well-rounded diet incorporating these macronutrients 
        in balanced proportions is crucial for maintaining energy, supporting metabolism, and ensuring overall 
        health. Consuming a variety of foods that provide all three macronutrients is essential for meeting 
        the body's nutritional requirements."
    </p>`,
    `<h1 class='step-title'>Micronutrients</h1>
    <p>
      Micronutrients are vital vitamins and minerals that the body requires in 
        minute quantities to maintain optimal health and physiological function. 
        Unlike macronutrients like carbohydrates, proteins, and fats, which supply energy, 
        micronutrients are crucial for supporting the immune system, facilitating normal growth 
        and development, and ensuring overall well-being. Key micronutrients include vitamins 
        such as A, C, D, E, K, and the B-complex group, along with minerals like iron, zinc, iodine, and calcium. 
        Insufficient intake of these nutrients can lead to various health issues, including anemia, 
        scurvy, and bone diseases, underscoring their essential role in a balanced diet. 
        A diverse diet rich in fruits, vegetables, grains, and animal products typically 
        provides the necessary levels of these critical nutrients.
    </p>`,
  ];

  const target2event = {
    0: () => {
      select("#chart3").attr("src", "src/assets/images/macro1.jpeg");
      select("#chart4").attr("src", "src/assets/images/macro2.jpeg");
    },
    1: () => {
      select("#chart3").attr("src", "src/assets/images/micro1.jpeg");
      select("#chart4").attr("src", "src/assets/images/micro2.jpeg");
    },
  };

  // Scroll iterator
  let value;

  // Trigger events on scroll
  $: if (typeof value !== "undefined") target2event[value]();
</script>

<h2 class="body-header">What are major components of a good diet?</h2>
<p class="body-text">
  Here's a breakdown of your main dietary needs: Macronutrients and Micronutrients
</p>
<section>
  <!-- Scroll container -->
  <div class="section-container">
    <div class="steps-container">
      <Scrolly bind:value>
        {#each steps as text, i}
          <div class="step" class:active={value === i}>
            <div class="step-content">{@html text}</div>
          </div>
        {/each}
        <div class="spacer" />
      </Scrolly>
    </div>
    <div class="charts-container">
      <div class="chart-one">
        <img id="chart3" src="src/assets/macro1.jpeg" alt="Chart 3 Image" />
      </div>
      <div class="chart-two">
        <img id="chart4" src="src/assets/macro2.jpeg" alt="Chart 4 Image" />
      </div>
    </div>
  </div>
  <!-- End scroll -->
  <br /><br />
  <p class="body-text">And that's the end of our scrolly.</p>
</section>

<style>
  #chart3,
  #chart4 {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  .chart-one,
  .chart-two {
    width: 100%;
    height: 100%;
  }
  .charts-container {
    position: sticky;
    top: 10%;
    display: grid;
    width: 50%;
    grid-template-columns: 100%;
    grid-row-gap: 2rem;
    grid-column-gap: 0rem;
    grid-template-rows: repeat(2, 1fr);
    height: 85vh;
    border: 3px solid black;
  }
  .section-container {
    margin-top: 1em;
    text-align: center;
    transition: background 100ms;
    display: flex;
  }
  .step {
    height: 110vh;
    display: flex;
    place-items: center;
    justify-content: center;
  }
  .step-content {
    font-size: 18px;
    background: var(--bg);
    color: #ccc;
    border-radius: 1px;
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    text-align: left;
    width: 75%;
    margin: auto;
    max-width: 500px;
    font-family: var(--font-main);
    line-height: 1.3;
    border: 5px solid var(--default);
  }
  .step.active .step-content {
    background: #f1f3f3ee;
    color: var(--squid-ink);
  }
  .steps-container {
    height: 100%;
  }
  .steps-container {
    flex: 1 1 40%;
    z-index: 10;
  }
  .section-container {
    flex-direction: column-reverse;
  }
  .steps-container {
    pointer-events: none;
  }
  .charts-container {
    top: 7.5%;
    width: 75%;
    margin: auto;
  }
  .step {
    height: 130vh;
  }
  .step-content {
    width: 65%;
    max-width: 768px;
    font-size: 17px;
    line-height: 1.6;
  }
  .spacer {
    height: 100vh;
  }
</style>
