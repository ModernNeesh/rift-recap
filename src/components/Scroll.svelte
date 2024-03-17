<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import SearchBar from "./SearchBar.svelte";
  import Line from "./line.svelte";
  import Bar from "./bar.svelte";
  import OppLine from "./opp_line.svelte";
  import { onMount } from "svelte";
  import * as d3 from 'd3';
  import Recap from "./Recap.svelte";

  import {fly, draw} from "svelte/transition";
  import {cubicInOut} from "svelte/easing";
  import {
    Card,
    CardBody,
    CardText,
  } from '@sveltestrap/sveltestrap';


  //Variables for champion counts, gold over time data, opponent data, and color scheme
  let champ_data;
  let gold_data;
  let opponent_data;
  let color;

  //Best and worst early, late, and overall champs
  let early;
  let late;
  let overall;

  //Hardest and easiest early, late, and overall matchups
  let early_counter;
  let late_counter;
  let overall_counter;


  //Strongest point in game against each champion
  let strongest_point = {};

  //List of worst overall matchups, in order
  let worst_matchups;

  //Player's champion pool, sorted by overall strength
  let pool;

  //What champion the player should play
  let champ_to_play_against = {};


  let champ_counters;


  let to_change = {'MonkeyKing' : 'Wukong',
                    'Nunu' : 'Nunu & Willump',
                    'Renata' : 'Renata Glasc',
                    'AurelionSol': 'Aurelion Sol',
                    'Belveth' : 'Bel\'Veth',
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

  let icon_map = Object.fromEntries(Object.entries(to_change).map(([key, value]) => [value, key]));
  let count, index, offset, progress;
  let width, height;

  function get_data(event){
    gold_data = event.detail.gold_data;
    champ_data = event.detail.champ_data;
    opponent_data = event.detail.opponent_data;
    color = event.detail.color;

    get_best_worst_data();

    

    get_champ_to_play();

  }

  let kda, damage_dealt_to_champ, damage_taken, match_time, critical_strike, items, runes;
  function get_stats(event){
    kda = event.detail.kda;
    damage_dealt_to_champ = event.detail.damage_dealt_to_champ;
    damage_taken = event.detail.damage_taken;
    match_time = event.detail.match_time;
    critical_strike = event.detail.critical_strike;
    items = event.detail.items;
    runes = event.detail.runes;
  }

  onMount(async () => {
    
    let champ_names_re = await fetch('champ_names.json');
    let champ_counters_re = await fetch('champ_counters.json');

    //Create a map for champion names to their icons
    let champ_names = new Set(Object.keys(await champ_names_re.json()));
    get_icon_map(champ_names);
    champ_counters = await champ_counters_re.json();
  })
  /*
    HELPER FUNCTIONS FOR LINE GRAPHS
    --------------------------------------

    These functions are used in both of the line graphs

  */
  function riemannSum(start, end, times, normalize = "none"){//Take starting index, ending index, and array of times to calculate riemann sum
        let normalizer = 1;
        
        if(normalize == "strong"){
          normalizer = (end - start)**2;
        }
        else if(normalize == "weak"){
          normalizer = (end - start);
        }
        if(start > times.length || start >= end){
            return null;
        }
        if(end - start == 1){
            return 0.5*(times[start] + times[end])
        }
        let sum = arr => arr.reduce((a, b) => a + b);
        return 0.5*(times[start] + 2*sum(times.slice(start+1,end)) + times[end]) / normalizer;
    }

  function calculateBestAndWorst(data, opponent = false){//Get the worst and best champs in the early and late game as well as overall
        let first_champ = data[0].champion;
        //Best and worst early, late and overall champs
        let early_game = {'min': first_champ, 'max' : first_champ};
        let late_game = {'min': first_champ, 'max' : first_champ};
        let overall = {'min': first_champ, 'max' : first_champ};

        //Maps champs to their "gold value", essentially a measure of strength at that stage defined by the integral (riemann sum) of gold with respect to time
        let early_game_gold = {};
        let late_game_gold = {};
        let overall_gold = {};


        let player_gold = [];

        let by_champs = d3.group(data, d => d.champion);
        
        if(opponent){
          player_gold = Array.from((d3.group(by_champs.get("Your Average"), d => d.gold)).keys());
        }

        by_champs.keys().forEach(champ => {
            let champ_gold = by_champs.get(champ).map((d) => d.gold);
            console.log(champ);
            console.log(champ_gold);
            //Gold values for mid, early, and late game
            early_game_gold[champ] = riemannSum(0, 14, champ_gold);
            late_game_gold[champ] = riemannSum(30, champ_gold.length-1, champ_gold, 'weak');
            overall_gold[champ] = riemannSum(0, champ_gold.length-1, champ_gold, 'strong');

            if(early_game_gold[champ]){
                if (early_game_gold[champ] > early_game_gold[early_game['max']] || !early_game_gold[early_game['max']]){//If the early game is better than the current best
                    early_game['max'] = champ;
                }
                if (early_game_gold[champ] < early_game_gold[early_game['min']] || !early_game_gold[early_game['min']]){//If the early game is worse than the current worst
                    early_game['min'] = champ;
                }
            }
            
            if(late_game_gold[champ]){
                if (late_game_gold[champ] > late_game_gold[late_game['max']] || !late_game_gold[late_game['max']]){//If the late game is better than the current best
                    late_game['max'] = champ;
                }
                if (late_game_gold[champ] < late_game_gold[late_game['min']] || !late_game_gold[late_game['min']]){//If the late game is worse than the current worst
                    late_game['min'] = champ;
                }
            }
            
            if(overall_gold[champ]){
                if (overall_gold[champ] > overall_gold[overall['max']] || !overall_gold[overall['max']]){//If the overall is better than the current best
                    overall['max'] = champ;
                }
                if (overall_gold[champ] < overall_gold[overall['min']] || !overall_gold[overall['min']]){//If the overall is worse than the current worst
                    overall['min'] = champ;
                }
            }

            if(opponent){
              strongest_point[champ] = get_max_diff(player_gold, champ_gold);
            }
        });

        if(opponent){
          worst_matchups = sort_by_gold(overall_gold);
        }
        else{
          pool = new Set(sort_by_gold(overall_gold));
        }
        return [early_game, late_game, overall];
    }

    function get_max_diff(player_arr, opponent_arr){
      var difference = player_arr.map(function(item, index) {
        return item - opponent_arr[index];
      });
      return difference.reduce((iMax, x, i, arr) => x > arr[iMax] ? i : iMax, 0) + 1;
    }

    function sort_by_gold(obj){
      let obj_entries = Object.entries(obj);
      obj_entries.sort(function(a, b) {
            return b[1] - a[1];
      });
          
      obj_entries = obj_entries.map((x) => x[0]);
      const index = obj_entries.indexOf("Your Average");
      if (index > -1) { // only splice array when item is found
        obj_entries.splice(index, 1); // 2nd parameter means remove one item only
      }
      return obj_entries;
    }

    function get_icon_map(champ_names){
      let changed_names = new Set(Object.keys(icon_map));
      let unchanged_names = champ_names.difference(changed_names);
      let unchanged_obj = Object.fromEntries(Array.from(unchanged_names).map(key => [key, key]));
      Object.assign(icon_map, unchanged_obj);
      icon_map['Leblanc'] = 'Leblanc';
      delete icon_map['LeBlanc'];
      icon_map['None'] = 'None';
    }

    function get_champ_to_play(){//For the top 3 worst counters (for which there is an available-to-play counter), get that counter
      for (let i in worst_matchups){
        let opp = worst_matchups[i];
        for(let j in champ_counters[opp]){
          let opp_counter = champ_counters[opp][j];
          if(pool.has(opp_counter)){
            champ_to_play_against[opp] = opp_counter;
            break;
          }
        }
        if(Object.keys(champ_to_play_against).length == 3){
          break;
        }
      };
    }

    function get_best_worst_data(){//Get the best and worst champions for the player as well as counters
      [early, late, overall] = calculateBestAndWorst(gold_data);
      [early_counter, late_counter, overall_counter] = calculateBestAndWorst(opponent_data, true);

      function replace_average(minmax){
        if(minmax['min'] == 'Your Average'){
          minmax['min'] = 'None';
        }
        if(minmax['max'] == 'Your Average'){
          minmax['max'] = 'None';
        }
      }

      replace_average(early_counter);
      replace_average(late_counter);
      replace_average(overall_counter);
    }
</script>

<main>
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
  {#if index+1==2 && offset>=0.2}
    <div>
      <Card theme="dark">
        <CardBody>
          <CardText>Search will take a while, please be patient and don't scroll!</CardText>
        </CardBody>
      </Card>
    </div>
  {/if}
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
    <section id="intro">
      {#if index!=1}
        <div class="description" transition:fly={{ delay: 0, duration: 300, x: 0, y: -20, opacity: 0.5, easing: cubicInOut }}>
          <h1>Welcome to Rift Recap</h1>
          <p>
            If you're a League of Legends player looking to enhance your skills, 
            we're here to offer some support. Our website provides personalized summaries 
            and visual representations of your gaming data, alongside evaluations of your 
            match performance and feedback that might be helpful. We hope to assist you in 
            recognizing your strengths and weaknesses, guiding you on a path to improvement. 
            Whether you play casually or competitively, we aim to provide insights that could 
            help you refine your strategy, improve your mastery of champions, and make your 
            overall gameplay experience a bit better.
          </p>
        </div>
      {/if}
    </section>
    <section id="searchbar">
      {#if index+1==2 & offset>=0.1}
        <div class="anim_wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0.5, easing: cubicInOut }}>
          <SearchBar {to_change} on:sending_data = {get_data} on:recap_data = {get_stats}/>
        </div>
      {/if}
    </section>
    <section id="recap">
      {#if kda && damage_dealt_to_champ && damage_taken && match_time && critical_strike && runes}
      {#if index+1==3 & offset>=0.2}
        <div class="anim_wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0.5, easing: cubicInOut }}>
          <Recap kda={kda} damage_dealt_to_champ={damage_dealt_to_champ} damage_taken={damage_taken} match_time={match_time} critical_strike={critical_strike} items={items} runes={runes}/>
        </div>
      {/if}
      {/if}
    </section>
    <section id="graph">
      {#if champ_data}
      {#if index+1==4 & offset>=0.2}
        <div class="anim_wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0.5, easing: cubicInOut }}>
          <Bar {champ_data} {color} />
        </div>
      {/if}
      {/if}
    </section>

    <section id="graph">
    {#if gold_data}
    {#if index+1==5 & offset>=0.2}
      <div class="anim_wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0.5, easing: cubicInOut }}>
        <Line {gold_data} {color} {icon_map}/>
      </div>
    {/if}
    {/if}
    </section>

    <section>
      {#if early}
      {#if index+1==6 & offset>=0.2}
        <h4 class = "header" transition:fly={{ delay: 0, duration: 100, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>Champion Pool Analysis</h4>

        <div class="performance-wrapper">

          <div class="champ-icon-wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>
            <div class="champ-icon">
              <img class = "icon" src = "champion_icons/{icon_map[early['min']]}.png" alt = "{early['min']}" width ="50" height = "50">
            </div>
            <div class="champ-icon">
              <img class = "icon" src = "champion_icons/{icon_map[early['max']]}.png" alt = "{early['max']}" width ="50" height = "50">
            </div>
            <div class="champ-icon">
              <img class = "icon" src = "champion_icons/{icon_map[late['min']]}.png" alt = "{late['min']}" width ="50" height = "50">
            </div>
            <div class="champ-icon">
              <img class = "icon" src = "champion_icons/{icon_map[late['max']]}.png" alt = "{late['max']}" width ="50" height = "50">
            </div>
            <div class="champ-icon">
              <img class = "icon" src = "champion_icons/{icon_map[overall['min']]}.png" alt = "{icon_map[overall['min']]}" width ="50" height = "50">
            </div>
            <div class="champ-icon">
              <img class = "icon" src = "champion_icons/{icon_map[overall['max']]}.png" alt = "{icon_map[overall['max']]}" width ="50" height = "50">
            </div>
          </div>

          <div class="champ-name-wrapper" transition:fly={{ delay: 200, duration: 500, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>
            <div class="champ-name">
              {early['min']}
            </div>
            <div class="champ-name">
              {early['max']}
            </div>
            <div class="champ-name">
              {late['min']}
            </div>
            <div class="champ-name">
              {late['max']}
            </div>
            <div class="champ-name">
              {overall['min']}
            </div>
            <div class="champ-name">
              {overall['max']}
            </div>
          </div>

          <div class="stage-wrapper">
            <div class="stage" transition:fly={{ delay: 400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Weakest Early Game Champion</p>
            </div>
            <div class="stage" transition:fly={{ delay: 600, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Strongest Early Game Champion</p>
            </div>
            <div class="stage" transition:fly={{ delay: 800, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Weakest Late Game Champion</p>
            </div>
            <div class="stage" transition:fly={{ delay: 1000, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Strongest Late Game Champion</p>
            </div>
            <div class="stage" transition:fly={{ delay: 1200, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Worst Champion Consistently</p>
            </div>
            <div class="stage" transition:fly={{ delay: 1400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Strongest Champion Consistently</p>
            </div>
          </div>

        </div>
        
      {/if}
      {/if}
    </section>

    <section id="graph">
      {#if opponent_data}
      {#if index+1==7}
        <div class="anim_wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0.5, easing: cubicInOut }}>
          <OppLine {opponent_data} {color} {icon_map}/>
        </div>
      {/if}
      {/if}  
    </section>

    <section>
      {#if early}
      {#if index+1==8 & offset>=0.2}
        <h4 class="header" transition:fly={{ delay: 0, duration: 100, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>Matchups Analysis</h4>
    
        <div class="performance-wrapper">
          <div class="champ-icon-wrapper" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>
            <img class="icon" src="champion_icons/{icon_map[early_counter['min']]}.png" alt="{early_counter['min']}" width="50" height="50">
            <img class="icon" src="champion_icons/{icon_map[early_counter['max']]}.png" alt="{early_counter['max']}" width="50" height="50">
            <img class="icon" src="champion_icons/{icon_map[late_counter['min']]}.png" alt="{late_counter['min']}" width="50" height="50">
            <img class="icon" src="champion_icons/{icon_map[late_counter['max']]}.png" alt="{late_counter['max']}" width="50" height="50">
            <img class="icon" src="champion_icons/{icon_map[overall_counter['min']]}.png" alt="{overall_counter['min']}" width="50" height="50">
            <img class="icon" src="champion_icons/{icon_map[overall_counter['max']]}.png" alt="{overall_counter['max']}" width="50" height="50">
          </div>
    
          <div class="champ-name-wrapper" transition:fly={{ delay: 200, duration: 500, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>
            <div class="champ-name">{early_counter['min']}</div>
            <div class="champ-name">{early_counter['max']}</div>
            <div class="champ-name">{late_counter['min']}</div>
            <div class="champ-name">{late_counter['max']}</div>
            <div class="champ-name">{overall_counter['min']}</div>
            <div class="champ-name">{overall_counter['max']}</div>
          </div>
    
          <div class="stage-wrapper">
            <div class="stage" transition:fly={{ delay: 400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Easiest Laning Against</p>
            </div>
            <div class="stage" transition:fly={{ delay: 600, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Hardest Laning Against</p>
            </div>
            <div class="stage" transition:fly={{ delay: 800, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Easiest Late Game Against</p>
            </div>
            <div class="stage" transition:fly={{ delay: 1000, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Hardest Late Game Against</p>
            </div>
            <div class="stage" transition:fly={{ delay: 1200, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Overall Best Against</p>
            </div>
            <div class="stage" transition:fly={{ delay: 1400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              <p>Overall Worst Against</p>
            </div>
          </div>
        </div>
      {/if}
      {/if}
    </section>
    

    <section id="improvement">
      {#if early}
      {#if index+1==9 & offset>=0.2}
        <h4 class = "header">How to Improve</h4>

        <div class="improvement-wrapper">

          <div class="champ-icon-wrapper-improve" transition:fly={{ delay: 0, duration: 300, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>
            <div class="champ-icon">
              {#if Object.keys(champ_to_play_against).length >= 1}
                <img class = "icon" src = "champion_icons/{icon_map[worst_matchups[0]]}.png" alt = "{early['min']}" width ="50" height = "50">
              {/if}
            </div>
            <div class="champ-icon">
              {#if Object.keys(champ_to_play_against).length >= 2}
                <img class = "icon" src = "champion_icons/{icon_map[worst_matchups[1]]}.png" alt = "{early['min']}" width ="50" height = "50">
              {/if}
            </div>
            <div class="champ-icon">
              {#if Object.keys(champ_to_play_against).length >= 3}
                <img class = "icon" src = "champion_icons/{icon_map[worst_matchups[2]]}.png" alt = "{early['min']}" width ="50" height = "50">
              {/if}
            </div>
          </div>

          <div class="champ-name-wrapper-improve" transition:fly={{ delay: 200, duration: 500, x: 0, y: -10, opacity: 0, easing: cubicInOut }}>
            <div class="champ-name-improve">
              {#if Object.keys(champ_to_play_against).length >= 1}
                {worst_matchups[0]}
              {/if}
            </div>
            <div class="champ-name-improve">
              {#if Object.keys(champ_to_play_against).length >= 2}
                {worst_matchups[1]}
              {/if}
            </div>
            <div class="champ-name-improve">
              {#if Object.keys(champ_to_play_against).length >= 3}
                {worst_matchups[2]}
              {/if}
            </div>
          </div>

          <div class="stage-wrapper-improve">
            {#if Object.keys(champ_to_play_against).length >= 1}
            <div class="stage" transition:fly={{ delay: 400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              To beat <text font-weight = "bold">{worst_matchups[0]}</text>:
              <span class = "li"> Play {champ_to_play_against[worst_matchups[0]]}, since they are the best counter to {worst_matchups[0]} 
                                  in your champion pool.</span>
              <span class = "li"> Try to play for a lead around the {strongest_point[worst_matchups[0]]} minute mark, since that is when you tend to be strongest in this matchup. </span>
            </div>
            {/if}
            {#if Object.keys(champ_to_play_against).length >= 2}
            <div class="stage" transition:fly={{ delay: 400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              To beat {worst_matchups[1]}:
              <span class = "li"> Play {champ_to_play_against[worst_matchups[1]]}, since they are the best counter to {worst_matchups[1]} 
                                  in your champion pool.</span>
              <span class = "li"> Try to play for a lead around the {strongest_point[worst_matchups[1]]} minute mark, since that is when you tend to be strongest in this matchup. </span>
            </div>
            {/if}
            {#if Object.keys(champ_to_play_against).length >= 3}
            <div class="stage" transition:fly={{ delay: 400, duration: 1000, x: 1000, y: 0, opacity: 0, easing: cubicInOut }}>
              To beat {worst_matchups[2]}:
              <span class = "li"> Play {champ_to_play_against[worst_matchups[2]]}, since they are the best counter to {worst_matchups[2]} 
                                  in your champion pool.</span>
              <span class = "li"> Try to play for a lead around the {strongest_point[worst_matchups[2]]} minute mark, since that is when you tend to be strongest in this matchup. </span>
            </div>
            {/if}
          </div>

        </div>
        
      {/if}
      {/if}
    </section>
  </div>

  </Scroller>
</main>


<style>
  @import 'https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css';
  @import url("https://fonts.googleapis.com/css2?family=Quicksand:wght@300..700&display=swap");

  main {
    font-family: "Quicksand", sans-serif;
  }

  #intro {
    display: flex;
    justify-content: center;
    text-align: center;
  }

  #recap {
    display: flex;
    justify-content: center;
    text-align: center;
  }

  .description{
    margin: 0 auto;
    height: auto;
    position: relative;
    padding: 1em;
  }

  .description h1{
    margin-top: 20%;
    margin-bottom: 10%;
  }

  .description p{
    text-align: justify;
  }

  #intro {
    display: flex;
    justify-content: center;
    text-align: center;
  }
  
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
    background-color:#ff99e4;
    background-image:
    radial-gradient(at 95% 62%, hsla(215,84%,65%,1) 0px, transparent 50%),
    radial-gradient(at 59% 78%, hsla(348,98%,70%,1) 0px, transparent 50%),
    radial-gradient(at 82% 36%, hsla(266,96%,73%,1) 0px, transparent 50%),
    radial-gradient(at 43% 16%, hsla(177,89%,62%,1) 0px, transparent 50%),
    radial-gradient(at 43% 68%, hsla(96,75%,76%,1) 0px, transparent 50%),
    radial-gradient(at 10% 30%, hsla(245,76%,68%,1) 0px, transparent 50%),
    radial-gradient(at 96% 93%, hsla(156,65%,67%,1) 0px, transparent 50%);
  }

  .foreground {
    width: 70%;
    margin: 0 auto;
    height: auto;
    position: relative;
    outline: 1px solid black;
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.2) /*  40% opaque */;
    visibility: hidden;
  }

  .header{
    font-size: 30 px;
    font-weight: bold;
  }

  section {
    height: 80vh;
    background-color: rgba(189, 207, 206, 0.212);
    text-align: center;
    max-width: 100%;
    color: black;
    padding: 1em;
  }

  #searchbar {
    display: flex;
    justify-content: center;
    text-align: center;
  }

  .icon {
    display: inline;
    position: relative;
  }

  .counter {
    width: 32%;
    text-align: left;
    font-size: 14 px;
    font-weight: bold;
    margin: 5em 5em 5em 5em;
  }


  span.li  {display: list-item; margin-left: 2em; text-align: top;}

  .anim_wrapper {
    display: flex;
    justify-content: center;
    text-align: center;
  }

  .performance-wrapper {
    display: flex;
    text-align: center;
    align-items: center;
  }

  .stage-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: start;
    text-align: left;
    gap: 43px;
    margin-left: 12%;
    margin-top: 5px;
    font-size: 25px;
    overflow: hidden;
  }

  .champ-icon-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: start;
    text-align: left;
    gap: 20px;
    margin-right: 10px;
    margin-left: 23%;
  }


  img.icon {
    width: 75px;
    height: 75px;
  }

  .champ-name-wrapper {
    display: flex;
    flex-direction: column;
    margin-top: -20px;
    text-align: left;
    gap: 45px;
  }


  .champ-name {
    display: flex;
    justify-content: left;
    text-align: center;
    margin-top: 15px;
    font-size: 25px;
  }



  .improvement-wrapper {
    display: flex;
    text-align: center;
    align-items: center;
  }

  .champ-icon-wrapper-improve{
    display: flex;
    flex-direction: column;
    justify-content: start;
    text-align: left;
    gap: 50px;
    margin-right: 10px;
    margin-left: 23%;
  }
  #improvement {
    height: 150vh;
  }

  :global(.card) {
    width: 15%;
    position: relative;
    top: 300px;
  }

  
  .champ-name-wrapper-improve{
    display: flex;
    flex-direction: column;
    margin-top: -20px;
    text-align: left;
    line-height: 700%;
  }

  .champ-name-improve {
    display: flex;
    justify-content: left;
    text-align: center;
    margin-top: 15px;
    font-size: 25px;
    width: 130px;
  }
  .stage-wrapper-improve{
    display: flex;
    flex-direction: column;
    justify-content: start;
    text-align: left;
    gap: 20px;
    margin-left: 12%;
    margin-top: 30px;
    font-size: 16px;
    overflow: hidden;
  }
</style>
