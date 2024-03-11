<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import SearchBar from "./SearchBar.svelte";
  import Line from "./line.svelte";
  import Bar from "./bar.svelte";
  import OppLine from "./opp_line.svelte";
  import { onMount } from "svelte";
  import * as d3 from 'd3';
  import Recap from "./Recap.svelte";


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

  let champ_names;

  let count, index, offset, progress;
  let width, height;
  let readme;
  let line;

  function get_data(event){
    gold_data = event.detail.gold_data;
    champ_data = event.detail.champ_data;
    opponent_data = event.detail.opponent_data;
    color = event.detail.color;

    [early, late, overall] = calculateBestAndWorst(gold_data);
    [early_counter, late_counter, overall_counter] = calculateBestAndWorst(opponent_data);

    function replace_average(minmax){
      if(minmax['min'] == 'Your Average'){
        minmax['min'] = 'none of the champions';
      }
      if(minmax['max'] == 'Your Average'){
        minmax['max'] = 'none of the champions';
      }
    }

    replace_average(early_counter);
    replace_average(late_counter);
    replace_average(overall_counter);
  }

  let position, kda, damage_dealt_to_champ, damage_taken, match_time, critical_strike, items, runes;
  function get_stats(event){
    position = event.detail.position;
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
    champ_names = new Set(Object.keys(await champ_names_re.json()));
    let changed_names = new Set(Object.keys(icon_map));
    let unchanged_names = champ_names.difference(changed_names);
    let unchanged_obj = Object.fromEntries(Array.from(unchanged_names).map(key => [key, key]));
    Object.assign(icon_map, unchanged_obj);
    icon_map['Leblanc'] = 'Leblanc';
    delete icon_map['LeBlanc'];
    icon_map['none of the champions'] = 'None';
  })
  /*
    HELPER FUNCTIONS FOR LINE GRAPHS
    --------------------------------------

    These functions are used in both of the line graphs

  */
  function riemannSum(start, end, times){//Take starting index, ending index, and array of times to calculate riemann sum
        if(start > times.length || start >= end){
            return null;
        }
        if(end - start == 1){
            return 0.5*(times[start] + times[end])
        }
        let sum = arr => arr.reduce((a, b) => a + b);
        return 0.5*(times[start] + 2*sum(times.slice(start+1,end)) + times[end]) / (end - start);
    }

  function calculateBestAndWorst(data){//Get the worst and best champs in the early and late game as well as overall
        let first_champ = data[0].champion;
        //Best and worst early, late and overall champs
        let early_game = {'min': first_champ, 'max' : first_champ};
        let late_game = {'min': first_champ, 'max' : first_champ};
        let overall = {'min': first_champ, 'max' : first_champ};

        //Maps champs to their "gold value", essentially a measure of strength at that stage defined by the integral (riemann sum) of gold with respect to time
        let early_game_gold = {};
        let late_game_gold = {};
        let overall_gold = {};


        let by_champs = d3.group(data, d => d.champion);
        by_champs.keys().forEach(champ => {
            let champ_gold = Array.from((d3.group(by_champs.get(champ), d => d.gold)).keys());
            //Gold values for mid, early, and late game
            early_game_gold[champ] = riemannSum(0, 14, champ_gold);
            late_game_gold[champ] = riemannSum(30, champ_gold.length-1, champ_gold);
            overall_gold[champ] = riemannSum(0, champ_gold.length-1, champ_gold);
            
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
        });
        console.log(overall_gold);
        return [early_game, late_game, overall];
    }

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
  </div>


  <div class="foreground" slot="foreground">
    <section id="firstSection">
      <SearchBar {to_change} on:sending_data = {get_data} on:recap_data = {get_stats}/></section>
    <section>
      {#if position && kda && damage_dealt_to_champ && damage_taken && match_time && critical_strike && runes}
        <Recap position={position} kda={kda} damage_dealt_to_champ={damage_dealt_to_champ} damage_taken={damage_taken} match_time={match_time} critical_strike={critical_strike} items={items} runes={runes}/>
      {/if}
    </section>
    <section>
      {#if champ_data}
        <Bar {champ_data} {color} />
      {/if}
    </section>

    <section>    
    {#if gold_data}
      <Line {gold_data} {color}/>
    {/if}
    </section>

    <section class = "midsection">
      <p class = "header">Strongest and Weakest champions</p>
      {#if early}
        <p>
        Your weakest early game champion is <img class = "icon" src = "champion_icons/{icon_map[early['min']]}.png" alt = "{early['min']}" width ="20" height = "20"> {early['min']}, 
        and your strongest early game champion is <img class = "icon" src = "champion_icons/{icon_map[early['max']]}.png" alt = "{early['max']}" width ="20" height = "20"> {early['max']}.


        Your weakest late game champion is <img class = "icon" src = "champion_icons/{icon_map[late['min']]}.png" alt = "{late['min']}" width ="20" height = "20"> {late['min']}, 
        and your strongest late game champion is <img class = "icon" src = "champion_icons/{icon_map[late['max']]}.png" alt = "{late['max']}" width ="20" height = "20"> {late['max']}.

        Your <img class = "icon" src = "champion_icons/{icon_map[overall['min']]}.png" alt = "{icon_map[overall['min']]}" width ="20" height = "20"> {overall['min']} gameplay is consistently the worst out of your champion pool, 
        while your <img class = "icon" src = "champion_icons/{icon_map[overall['max']]}.png" alt = "{icon_map[overall['max']]}" width ="20" height = "20"> {overall['max']} gameplay is consistently the strongest.
        </p>
      {/if}
    </section>

    <section>
      {#if opponent_data}
        <OppLine {opponent_data} {color}/>
      {/if}  
    </section>

    <section class = "midsection">
      <p class = "header">Easiest and Hardest Lane matchups</p>
      {#if early}
        <p>
        Your easiest laning phase matchup is  <img class = "icon" src = "champion_icons/{icon_map[early_counter['min']]}.png" alt = "{early_counter['min']}" width ="20" height = "20"> {early_counter['min']}, 
        and your hardest laning phase matchup is <img class = "icon" src = "champion_icons/{icon_map[early_counter['max']]}.png" alt = "{early_counter['max']}" width ="20" height = "20"> {early_counter['max']}.

        You do the best in the late game against <img class = "icon" src = "champion_icons/{icon_map[late_counter['min']]}.png" alt = "{late_counter['min']}" width ="20" height = "20"> {late_counter['min']}, 
        and you struggle the most in the late game against <img class = "icon" src = "champion_icons/{icon_map[late_counter['max']]}.png" alt = "{late_counter['max']}" width ="20" height = "20"> {late_counter['max']}.

        Overall, you have the easiest time against <img class = "icon" src = "champion_icons/{icon_map[overall_counter['min']]}.png" alt = "{icon_map[overall_counter['min']]}" width ="20" height = "20"> {overall_counter['min']}, 
        while you get countered hardest by <img class = "icon" src = "champion_icons/{icon_map[overall_counter['max']]}.png" alt = "{icon_map[overall_counter['max']]}" width ="20" height = "20"> {overall_counter['max']}.
        </p>
      {/if}
    </section>

    <section>
      <p class = "header">How To Improve</p>

      <p class = "counter">In order to beat <img class = "icon" src = "champion_icons/Irelia.png" alt = "Sett" width ="20" height = "20"> Irelia:
        <span class=li>Play <img class = "icon" src = "champion_icons/Sett.png" alt = "Sett" width ="20" height = "20">Sett, since Sett is the hardest counter to Irelia in your champion pool.</span>
        <span class=li>Try to play for the lead more around the 3 minute mark, as that is when you tend to have the biggest advantage.</span>
      </p>
      <p class = "counter">In order to beat <img class = "icon" src = "champion_icons/Tryndamere.png" alt = "Sett" width ="20" height = "20"> Tryndamere:
        <span class=li>Play <img class = "icon" src = "champion_icons/Sett.png" alt = "Sett" width ="20" height = "20">Sett, since Sett is the hardest counter to Tryndamere in your champion pool.</span>
        <span class=li>Try to play for the lead more around the 39 minute mark, as that is when you tend to have the biggest advantage.</span>
      </p>
      <p class = "counter">In order to beat <img class = "icon" src = "champion_icons/Fiora.png" alt = "Sett" width ="20" height = "20"> Fiora:
        <span class=li>Play <img class = "icon" src = "champion_icons/Sett.png" alt = "Sett" width ="20" height = "20">Sett, since Sett is the hardest counter to Fiora in your champion pool.</span>
        <span class=li>Try to play for the lead more around the 3 minute mark, as that is when you tend to have the biggest advantage.</span>
      </p>
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
    background-color: rgb(126, 130, 145);
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
    color: rgb(41, 49, 65);
  }

  .header{
    font-size: 30 px;
    font-weight: bold;
  }

  section {
    height: 100vh;
    background-color: rgb(174, 179, 197); /* 20% opaque */
    /* color: white; */
    text-align: center;
    max-width: 100%; 
    color: rgb(30, 29, 29);
    padding: 0.5em;
    margin: 0 0 0.5em 0;
    font-size: 20 px;
    line-height: 200%;
    font-weight: bold;
    
  }

  .midsection {
    height: 50vh;
    background-color: rgb(174, 179, 197); /* 20% opaque */
    /* color: white; */
    text-align: center;
    max-width: 100%; 
    color: rgb(30, 29, 29);
    padding: 0.5em;
    margin: 0 0 0.5em 0;
    font-size: 20 px;
    line-height: 200%;
    font-weight: bold;
    
  }

  #firstSection {
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


  span.li  {display: list-item; margin-left: 2em}

  
</style>
