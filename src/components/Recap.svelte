<script>
  export let kda;
  export let damage_dealt_to_champ;
  export let damage_taken;
  export let match_time;
  export let critical_strike;
  export let items;
  export let runes;

  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  // import data
  // import runes_data from '../data/runes_flattened.json';
  // import valid_item_icons from '../data/valid_item_icons.json';


  let legendary_items;
  let valid_item_icons;
  let most_picked_5_items;
  let runes_data;
  let most_picked_5_runes;
  let isDataLoaded = false; // Track if data has been loaded
  onMount(async () => {
    const runesDataResponse = await fetch('runes_flattened.json');
    runes_data = await runesDataResponse.json();
    const validItemIconsResponse = await fetch('valid_item_icons.json');
    valid_item_icons = await validItemIconsResponse.json();
    
    legendary_items = Object.keys(items).filter(key => valid_item_icons.includes(key)).reduce((acc, key) => {
      acc[key] = items[key];
      return acc;
    }, {});

    most_picked_5_items = Object.entries(legendary_items).sort((a, b) => b[1] - a[1]).slice(0, 5);
    most_picked_5_runes = Object.entries(runes).sort((a, b) => b[1] - a[1]).slice(0, 5);

    isDataLoaded = true;
  });

  // process data
  let most_kill = kda.sort((a, b) => b.kills - a.kills)[0];
  let most_death = kda.sort((a, b) => b.deaths - a.deaths)[0];
  let most_assist = kda.sort((a, b) => b.assists - a.assists)[0];
  let most_damage_dealt_to_champ = Object.entries(damage_dealt_to_champ).sort((a, b) => b[1] - a[1])[0];
  let most_damage_taken = Object.entries(damage_taken).sort((a, b) => b[1] - a[1])[0];
  let longest_match_time = Object.entries(match_time).sort((a, b) => b[1] - a[1])[0];
  let best_critical_strike = Object.entries(critical_strike).sort((a, b) => b[1] - a[1])[0];



  // console.log(most_kill);
  // console.log(most_death);
  // console.log(most_assist);
  // console.log(most_damage_dealt_to_champ);
  // console.log(most_damage_taken);
  // console.log(longest_match_time);
  // console.log(best_critical_strike);
  // console.log(most_picked_5_items);
  // console.log(most_picked_5_runes);




</script>

<main>
  <div class="recap">
    <h2>Recap</h2>
    <div class="match_highlights">
      <div class="highlights_wrapper">
        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{most_kill.champ}" src={`champion_icons/${most_kill.champ}.png`} />
          <p>Most kills</p>
          <p>{`${most_kill.kills}/${most_kill.deaths}/${most_kill.assists}`}</p>
        </div>

        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{most_death.champ}" src={`champion_icons/${most_death.champ}.png`} />
          <p>Most deaths</p>
          <p>{`${most_death.kills}/${most_death.deaths}/${most_death.assists}`}</p>
        </div>

        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{most_assist.champ}" src={`champion_icons/${most_assist.champ}.png`} />
          <p>Most assists</p>
          <p>{`${most_assist.kills}/${most_assist.deaths}/${most_assist.assists}`}</p>
        </div>

        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{most_damage_dealt_to_champ[0]}" src={`champion_icons/${most_damage_dealt_to_champ[0]}.png`} />
          <p>Damage dealt</p>
          <p>{most_damage_dealt_to_champ[1]} dmg</p>
        </div>

        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{most_damage_taken[0]}" src={`champion_icons/${most_damage_taken[0]}.png`} />
          <p>Damage taken</p>
          <p>{most_damage_taken[1]} dmg</p>
        </div>

        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{best_critical_strike[0]}" src={`champion_icons/${best_critical_strike[0]}.png`} />
          <p>Best critical strike</p>
          <p>{best_critical_strike[1]} dmg</p>
        </div>

        <div class="champ_icon_wrapper">
          <img id="champ_icon" alt="{longest_match_time[0]}" src={`champion_icons/${longest_match_time[0]}.png`} />
          <p>Longest match</p>
          <p>{Math.floor(longest_match_time[1]/60)}m {longest_match_time[1]%60}s</p>
        </div>
      </div>
    </div>

    <div class="most_picked_items">
      <h5>MOST PICKED ITEMS</h5>
      {#if most_picked_5_items}
        <div class="item_wrapper">
          {#each most_picked_5_items as item, i}
            <div class={`item${i}`}>
              <img id="item_icon" alt="{item[0]}" src={`item_icons/${item[0]}.png`} />
              <p>{item[1]}</p>
            </div>
          {/each}
        </div>
      {/if}
    </div>

    <div class="most_picked_runes">
      <h5>MOST PICKED RUNES</h5>
      {#if most_picked_5_runes}
        <div class="rune_wrapper">
          {#each most_picked_5_runes as rune, i}
            <div class={`rune${i}`}>
              <img id="rune_icon" alt="{runes_data[rune[0]]}" src={`rune_icons/${runes_data[rune[0]]}.png`} />
              <p>{rune[1]}</p>
            </div>
          {/each}
        </div>
      {/if}
    </div>
  </div>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300..800;1,300..800&display=swap');

  
  :root {
    font-family: "Open Sans", sans-serif;
  }

  
  .recap {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .match_highlights {
    display: flex;
    width: 90%;
    justify-content: center;
    margin-bottom: 5%;
  }

  .highlights_wrapper {
    display: flex;
    flex-direction: row;
    justify-content: center;
    gap: 20px;
    align-items: center;
  }

  .champ_icon_wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 10px;
  }

  .most_picked_items {
    width: 90%;
    display: flex;
    justify-content: center;
    flex-direction: column;
    margin-bottom: 5%;
  }

  .most_picked_runes {
    width: 90%;
    display: flex;
    justify-content: center;
    flex-direction: column;
  }

  .item_wrapper {
    display: flex;
    justify-content: center;
    flex-direction: row;
    gap: 20px;
  }

  .rune_wrapper {
    display: flex;
    justify-content: center;
    flex-direction: row;
    gap: 20px;
  }


  #champ_icon {
    width: 60px;
    height: 60px;
  }

  #rune_icon {
    width: 60px;
    height: 60px;
  }

  #item_icon {
    width: 60px;
    height: 60px;
  }

  div.champ_icon_wrapper p {
    margin: 0px;
    white-space: nowrap;
    font-size: small;
  }


  
</style>