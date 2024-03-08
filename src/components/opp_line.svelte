<script>
    import * as d3 from 'd3';

    export let opponent_data;
    export let color;


    const width = 1100;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 40;

    let svg_line;
    let svg_legend;
    let lines;
    let points;

    // Placeholders for the axis elements.
    let gx;
    let gy;

    let checked_champs = {};
    let select_deselect_all = true;

    // map time to x-coordinate on screen
    let x = d3
        .scaleLinear()
        .domain(d3.extent(opponent_data, (d) => d.time))
        .range([marginLeft, width - marginRight]);
    // map gold to y-coordinate on screen
    let y = d3
        .scaleLinear()
        .domain(d3.extent(opponent_data, (d) => d.gold))
        .range([height - marginBottom, marginTop]);

    // make line
    $: line = d3
        .line()
        .x((d) => x(d.time)).y((d) => y(d.gold));

    // group data by champion
    let grouped;
    grouped = d3.group(opponent_data, (d) => d.champion);
    grouped.forEach(d => {
        checked_champs[d[0].champion] = true;
    });

    // x-axis
    $: gx = d3
        .select(gx)
        .call(d3.axisBottom(x).ticks(width / 80));

    // y-axis
    $: gy = d3.select(gy)
        .call(d3.axisLeft(y).ticks(null))
        .call(
            (g) => g.selectAll('.tick line')
            .clone()
            .attr('x2', width - marginRight - marginLeft)
            .attr('stroke-opacity', (d) => (d === 0 ? 1 : 0.1))
        );


    function uncheck_all(){
        if (select_deselect_all === true){
            for (let key in checked_champs){
                checked_champs[key] = false;
            }
            select_deselect_all = false;
        } else {
            for (let key in checked_champs){
                checked_champs[key] = true;
            }
            select_deselect_all = true;
        }
    }

    function uncheck_champ(d){
        checked_champs[d[0]] = !checked_champs[d[0]];
    };


    // interactive tooltip
    let tooltipPt = null;

    // calculate x,y coordinates of all data points
    let coords = [];
    for (let i = 0; i < opponent_data.length; i++){
        coords.push([x(opponent_data[i].time), y(opponent_data[i].gold)]);
    }
    
    function onPointerMove(event) {
        // find the closest point to the pointer, euclidean distance
        let pointer = d3.pointer(event);
        let min_dist = 100000;
        let min_index = 0;
        for (let i = 0; i < coords.length; i++){
            let dist = Math.sqrt((pointer[0] - coords[i][0])**2 + (pointer[1] - coords[i][1])**2);
            if (dist < min_dist){
                min_dist = dist;
                min_index = i;
            }
        }
        // only set tooltipPt if checkbox is checked
        if (checked_champs[opponent_data[min_index].champion] === true){
            tooltipPt = opponent_data[min_index];
        }
        //console.log(tooltipPt)

        /*
            show line if that line's champ attribute matches tooltipPt.champion,
            set opacity of other lines to 0.1.
        */
        if (tooltipPt){
            d3.selectAll("path").attr("stroke-opacity", function() {
                // dont change the opacity of the y-axis line
                if (!d3.select(this).classed("domain")){
                    const champAttribute = d3.select(this).attr("champ");
                    if (champAttribute === tooltipPt.champion && checked_champs[champAttribute] === true) {
                        return 0.75;
                    } else {
                        return 0.1;
                    }
                }
            });
        }


        /*
            show circle closest to pointer if that circle's champ attribute matches tooltipPt.champion,
            set opacity of other circles to 0.
        */
        if (tooltipPt){
            d3.selectAll("circle").attr("opacity", function() {
                if (!d3.select(this).classed("button")) {
                    const champAttribute = d3.select(this).attr("champ");
                    if (champAttribute === tooltipPt.champion && checked_champs[champAttribute] === true) {
                        return 1;
                    } else {
                        return 0;
                    }
                }
            });
        }
    }



    function onPointerLeave(event) {
        tooltipPt = null;
        d3.selectAll("path").attr("stroke-opacity", 0.75);
        d3.selectAll("circle").attr("opacity", 0);
        d3.selectAll(".button").attr("opacity", 1);
    }
    
    $: d3.select(svg_line)
        .on('pointermove pointerenter', onPointerMove)
        .on('pointerleave', onPointerLeave);

</script>

<main>
    <div class="line-plot">
            
        <svg
            bind:this={svg_line}
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
                    Minutes
                </text>
            </g>
            
            <!-- y-axis -->
            <g bind:this={gy} transform="translate({marginLeft},0)">
                <text
                    x="5"
                    y="{marginTop}"
                    font-weight="bold"
                    fill="black"
                    text-anchor="start"
                >
                    Earned Gold
                </text>
            </g>
            
        <!-- lines -->
        <g bind:this={lines}>
            {#each grouped as d, i}
                <path
                    key={i}
                    d={line(d[1])}
                    stroke={d[0] === "Your Average" ? "black" : color(d[0])}
                    stroke-width={checked_champs[d[0]] === true ? "2": "0"}
                    fill="none"
                    champ={d[0]}
                    stroke-opacity=.75
                />
            {/each}
        </g>
            
            <!-- points -->
            <g bind:this={points} stroke="#000" stroke-opacity="0.2">
                {#each opponent_data as d, i}
                    <circle
                        key={i}
                        cx={x(d.time)}
                        cy={y(d.gold)}
                        r="1.5"
                        fill="black"
                        opacity="0"
                        champ={d.champion}
                    />
                {/each}
            </g>

            <!-- tooltip -->
            {#if tooltipPt}
            <g transform="translate({x(tooltipPt.time)},{y(tooltipPt.gold)-50})" text-anchor="middle">
                <text 
                    font-size="12px"
                    dy="0"
                    opacity={checked_champs[tooltipPt.champion] === true ? "1": "0"}
                    font-weight="500"
                >
                    {tooltipPt.champion}
                </text>

                <text
                    font-size="12px"
                    dy="1.2em"
                    opacity={checked_champs[tooltipPt.champion] === true ? "1": "0"}
                    font-weight="500"
                >
                    {tooltipPt.time} minutes
                </text>

                <text 
                    font-size="12px" 
                    dy="2.4em"
                    opacity={checked_champs[tooltipPt.champion] === true ? "1": "0"}
                    font-weight="500"
                >
                    {Math.round(tooltipPt.gold)} gold
                </text>
            </g>
            {/if}

            
        </svg>

        <svg
            bind:this={svg_legend}
            width={200}
            {height}
            viewBox="0 0 {200} {height}"
            style="max-width: 100%; height: auto;"
        >
            <!-- legend -->
            <g transform="translate(10, 0)">
                <text
                    y={marginTop}
                    font-size="12"
                    font-weight="bold"
                    fill="black"
                    text-anchor="start"
                >
                    Champion
                </text>

                {#each grouped as d, i}
                    <g transform="translate(0, {40 + i * 20} )">
                        <rect
                            class="button"
                            width="10"
                            height="10"
                            fill={d[0] === "Your Average" && checked_champs[d[0]] === true ? "black" : checked_champs[d[0]] === true ? color(d[0]) : "white"}
                            stroke = "black"
                            id = "{d} rect" 
                            on:click = {uncheck_champ(d)}
                        />
                        <text
                            x="15"
                            y="10"
                            font-size="12"
                            font-weight="auto"
                            fill="black"
                            text-anchor="start"
                        >
                            {d[0]}
                        </text>
                    </g>
                {/each}
                
                <!-- Select / Deselect All -->
                <g transform = "translate(5, {50 + grouped.size*20} )">
                    <circle
                        class="button"
                        r="10" 
                        stroke = "black" 
                        fill={select_deselect_all === true ? "lightblue": "white"}
                        on:click = {uncheck_all}
                    />

                    <text
                        x="15"
                        y="5"
                        font-size="12"
                        font-weight="auto"
                        fill="black"
                        text-anchor="start"
                    >
                        Select / Deselect All
                    </text>
                </g>

            </g>

            

        </svg>
    </div>
    
</main>

<style>

:root {
    user-select: none;
}



.button {
    cursor: pointer;
}

</style>