<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let champ_data;
    export let color;

    //Add for debugging
    //let champ_data = {'Yone': 1, 'Aatrox': 3, 'Sett' : 4, 'Darius' : 3, 'Chogath' : 2, 'Briar' : 3} ;
    //let color;

    // set the dimensions and margins of the graph
    const width = 1100;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 50;

    let svg_bar;
    let svg_icons;
    let bars;

    // Placeholders for the axis elements.
    let gx;
    let gy;
    
    
    // Add X axis
    let x = d3
        .scaleLinear()
        .domain([0, d3.max(Object.values(champ_data)) + 1])
        .range([marginLeft + 50, width - marginRight]);
    // Y axis
    let y = d3
            .scaleBand()
            .range([marginBottom, height - marginTop - 10])
            .domain(Object.keys(champ_data))
            .padding(.1);
    // x-axis
    $: gx = d3
        .select(gx)
        .call(d3.axisBottom(x));

    // y-axis
    $: gy = d3.select(gy)
        .call(d3.axisLeft(y));
    //Add for debugging
    //$: color = d3.scaleOrdinal(d3.schemeTableau10).domain(Object.values(champ_data));
    
    
</script>

<main>
  <div class = 'bar_graph'>

    <svg
    bind:this={svg_bar}
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
    >
            <!-- x-axis -->
            <g 
                
                bind:this={gx}
                transform="translate(0,{height - marginBottom})" 
            >
                <text
                    x={width-70}
                    y="{-10}"
                    dy="0.32em"
                    fill="#000"
                    font-weight="bold"
                    text-anchor="start"
                >
                    Games Played
                </text>
            </g>
            
            <!-- y-axis -->
            <g bind:this={gy} transform="translate({marginLeft + 50},0)">
            </g>
        <!-- lines -->
        <g bind:this={bars}>
          {#each Object.entries(champ_data) as champ}
              <rect
                x = {x(0)}
                y = {y(champ[0])}
                style:width = {(x(champ[1]) - (marginLeft + 50))}
                height = {y.bandwidth()}
                fill = {color(champ[0])}
              >
                </rect>
          {/each}
        </g>
    </svg>


  </div>
</main>

<style>
  rect {
		transition: width 1s;
		position: absolute;
	}

  .bar_graph {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0px;
    padding: 0px;
  }
</style>