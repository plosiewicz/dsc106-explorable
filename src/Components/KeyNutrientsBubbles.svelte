<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let data = [
      { name: 'Protein', description: 'Essential for building and repairing tissues, and producing enzymes and hormones.' },
      { name: 'Carbohydrates', description: 'The body\'s main source of energy, but excessive consumption can lead to weight gain.' },
      { name: 'Fats', description: 'Necessary for energy and cell function, but unhealthy fats can increase the risk of heart disease.' },
      { name: 'Vitamins and Minerals', description: 'Vital for immune function, bone health, and overall metabolism.' }
    ];
  
    let selectedNutrient = null;
      
    onMount(() => {
      drawBubbleChart();
    });
  
    function drawBubbleChart() {
      const width = 800;
      const height = 400;
      const margin = { top: 20, right: 20, bottom: 50, left: 20 };
  
      // Clear any existing SVG
      d3.select('#bubble-chart').selectAll('*').remove();
  
      // Set up the SVG container
      const svg = d3.select('#bubble-chart')
        .append('svg')
        .attr('width', width)
        .attr('height', height);
  
      const bubblesGroup = svg.append('g')
        .attr('class', 'bubbles-group');
  
      const bubbleRadiusScale = d3.scaleLinear()
        .domain([0, data.length])
        .range([10, 40]);
  
      const simulation = d3.forceSimulation(data)
        .force('x', d3.forceX().strength(0.05).x((d, i) => (i + 1) * (width / (data.length + 1))))
        .force('y', d3.forceY().strength(0.5).y(height / 2))
        .force('collide', d3.forceCollide().radius(d => bubbleRadiusScale(1) + 5))
        .on('tick', () => {
          bubblesGroup.selectAll('.bubble')
            .attr('cx', d => d.x)
            .attr('cy', d => d.y);
        });
  
      const bubbles = bubblesGroup.selectAll('.bubble')
        .data(data)
        .enter()
        .append('circle')
        .attr('class', 'bubble')
        .attr('r', (d, i) => bubbleRadiusScale(i))
        .style('fill', '#FF6F61')
        .style('opacity', 0.7)
        .on('mouseover', (event, d) => {
          d3.select(event.target).style('opacity', 1);
        })
        .on('mouseout', (event, d) => {
          d3.select(event.target).style('opacity', 0.7);
        })
        .on('click', (event, d) => {
          selectedNutrient = d;
        });
  
      const labels = bubblesGroup.selectAll('.bubble-label')
        .data(data)
        .enter()
        .append('text')
        .attr('class', 'bubble-label')
        .attr('text-anchor', 'middle')
        .attr('dy', '.35em')
        .style('fill', 'white')
        .text(d => d.name);
  
      labels.each((d, i, nodes) => {
        const label = d3.select(nodes[i]);
        const labelWidth = label.node().getComputedTextLength();
        label.attr('x', (i + 1) * (width / (data.length + 1)) - labelWidth / 2);
        label.attr('y', height / 2);
      });
    }
  </script>
  
  <style>
    .bubble {
      cursor: pointer;
    }
  
    .bubble-label {
      font-size: 14px;
      pointer-events: none;
    }
  </style>
  
  <section>
    <h2><strong>Let's Explore Some Key Nutrients:</strong></h2>

    <div id="bubble-chart"></div>
  </section>
  
  {#if selectedNutrient}
  <section>
    <h3>{selectedNutrient.name}</h3>
    <p>{selectedNutrient.description}</p>
  </section>
  {/if}
  