<script>
  
  import { onMount } from "svelte";

  const api_key = "RGAPI-04433599-4dda-43b9-b870-b80519c77c6c";
  const puuid = "i2wJQUrDJMzLIBKc2cd9oe8p1_oLSyx90HrWlqUPdhoBYOVBkZcyxIgqaOYQFDThQ3CfcFAI80UIhw";


  /*
  async function get_match_gold(match_id, puuid){
    //Get API data for timeline and overall match data
    let timeline;
    let match;
    await fetch(`https://americas.api.riotgames.com/lol/match/v5/matches/${match_id}/timeline?api_key=${api_key}`)
      .then(response => response.json())
      .then(json => timeline = json['info']);
    await fetch(`https://americas.api.riotgames.com/lol/match/v5/matches/${match_id}?api_key=${api_key}`)
      .then(response => response.json())
      .then(json => match = json['info']);
    

    //Get the participant ID of the given person
    let participant = await get_participant(timeline['participants'], puuid);

    let champ_names = {};

    //Map each player's puuid in the match to the champion they were playing
    match['participants'].forEach(player =>{
      champ_names[player['puuid']] = player['championName']
    });

    let times = {};
    
    //Create the data for the player's gold over time for this game
    timeline['frames'].forEach(frame => {
      times[[champ_names[puuid], Math.round(frame['timestamp'] / 60000)]] = [frame['participantFrames'][`${participant}`]['totalGold']];
    })
    return times;
  }
  onMount(async () => {
    let matches;
    await fetch(`https://americas.api.riotgames.com/lol/match/v5/matches/by-puuid/${puuid}/ids?start=0&count=10&api_key=${api_key}`)
          .then(response => response.json())
          .then(json => matches = json);//Get most recent 10 matches 

    gold_over_time = await get_match_gold(matches[0], puuid);//Create an object for the data for gold over time

    matches.slice(1).forEach(async match => {
      let match_times = await get_match_gold(match, puuid);
      for (let key in match_times) { 
        if (gold_over_time.hasOwnProperty(key)) { 
          gold_over_time[key] = gold_over_time[key].concat(match_times[key]); 
        } else { 
          gold_over_time[key] = match_times[key]; 
        } 
      } 
    });
    console.log(gold_over_time);
    
  });
  */
</script>

<style>
  /* Write your CSS here */
</style>
