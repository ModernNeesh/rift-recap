<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import SearchBar from "./SearchBar.svelte";
  import Line from "./Line.svelte";
  import Bar from "./Bar.svelte";
  import { onMount } from "svelte";

  //Variables for champion counts, gold over time data, and color scheme
  let champ_data;
  let gold_data;
  let color;

  let to_change = {'MonkeyKing' : 'Wukong',
                    'Nunu' : 'Nunu & Willump',
                    'Renata' : 'Renata Glasc',
                    'AurelionSol': 'Aurelion Sol',
                    'Chogath': 'Cho\'Gath',
                    'DrMundo': 'Dr. Mundo',
                    'JarvanIV': 'Jarvan IV',
                    'Kaisa': 'Kai\'Sa',
                    'Khazix': 'Kha\'Zix',
                    'KogMaw': 'Kog\'Maw',
                    'KSante': 'K\'Sante',
                    'LeeSin': 'Lee Sin',
                    'MasterYi': 'Master Yi',
                    'MissFortune': 'Miss Fortune',
                    'RekSai' : 'Rek\'Sai',
                    'TahmKench' : 'Tahm Kench',
                    'TwistedFate': 'Twisted Fate',
                    'XinZhao' : 'Xin Zhao'};

  let count, index, offset, progress;
  let width, height;
  let readme;
  let line;

  function get_data(event){
    gold_data = event.detail.gold_data;
    champ_data = event.detail.champ_data;
    color = event.detail.color;
    console.log(champ_data);
  }

  onMount(async () => {
    const read = await fetch('readme_prototype.txt');
    readme = await read.text();
  });

</script>

<main>
  <header>

  </header>
  <Scroller
    top={0.0}
    bottom={1}
    threshold={0.5}
    bind:count
    bind:index
    bind:offset
    bind:progress
  >
  <div
    class="background"
    slot="background"
    bind:clientHeight={height}
    bind:clientWidth={width}
  >
    <div class="progress-bars">
      <p>Currect Section: <strong>{index+1}/{count}</strong></p>
      <progress value={count ? (index+1)/count : 0} />

      <p>offset in currect section</p>
      <progress value={offset || 0} />

      <p>total progress</p>
      <progress value={progress || 0} />
    </div>
  </div>


  <div class="foreground" slot="foreground">
    <section id="firstSection"><SearchBar {to_change} on:sending_data = {get_data}/></section>
    <section>
      {#if champ_data}
        <Bar {champ_data} {color}/>
      {/if}
      </section>
    <section>    
    {#if gold_data}
      <Line {gold_data} {color}/>
    {/if}
    </section>
    <section>
      <img id = 'icon' src="champion_icons/Aatrox.png" alt='aatrox'>
    </section>
  </div>

  </Scroller>
</main>


<style>
  @import 'https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css';
  
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
    outline: red solid 3px;
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.2) /*  40% opaque */;
    visibility: visible;
  }

  section {
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: magenta solid 3px;
    text-align: center;
    max-width: 100%; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  #firstSection {
    display: flex;
    justify-content: center;
  }


  
</style>
