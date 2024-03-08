<script>
  import {
    Dropdown,
    DropdownItem,
    DropdownMenu,
    DropdownToggle,
    InputGroup,
    InputGroupText,
    Input,
    Spinner
  } from '@sveltestrap/sveltestrap';
  import {createEventDispatcher} from 'svelte';
  import * as d3 from 'd3';

  export let to_change;

  const dispatch = createEventDispatcher();

  let region;
  let gameName;
  let tagLine;
  let loading = false;
  const NUM_MATCHES = 20;
  
  const validRegions = ["americas", "asia", "europe", "sea"];


  /*
    API CALL FUNCTIONS
    --------------------

    The below functions all just call the api for specific requests
  */

  async function get_puuid_by_name_tag(region, gameName, tagLine) {
    // Construct the API URL with the appropriate path parameters
    const apiUrl = `https://4i6m73advi.execute-api.us-west-1.amazonaws.com/rgapi/account/${region}/${gameName}/${tagLine}`;

    try {
      let response = await fetch(apiUrl, {
        headers: {
          'Content-Type': 'application/json',
        },
        mode: 'cors'
      });
        
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      let json = await response.json();
      return json.puuid;
    } catch (error) {
      console.error("Fetch error: ", error.message);
      // handle the error
    }
  }

  async function get_most_recent_match_ids_by_puuid(region, puuid, num_matches) {
    // Construct the API URL with the appropriate path parameters
    const apiUrl = `https://4i6m73advi.execute-api.us-west-1.amazonaws.com/rgapi/match_id/${region}/${puuid}/${num_matches}`;

    try {
      let response = await fetch(apiUrl, {
        headers: {
          'Content-Type': 'application/json',
        },
        mode: 'cors'
      });
        
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      let json = await response.json();
      return json;
    } catch (error) {
      console.error("Fetch error: ", error.message);
      // handle the error
    }
  }

  async function get_match_detail_by_match_id(region, match_id) {//Get match details for a match_id
    // Construct the API URL with the appropriate path parameters
    const apiUrl = `https://4i6m73advi.execute-api.us-west-1.amazonaws.com/rgapi/match_detail/${region}/${match_id}`;

    try {
      let response = await fetch(apiUrl, {
        headers: {
          'Content-Type': 'application/json',
        },
        mode: 'cors'
      });
        
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      let json = await response.json();
      return json.info;
    } catch (error) {
      console.error("Fetch error: ", error.message);
      // handle the error
    }
  }

  async function get_match_timeline_by_match_id(region, match_id) {//Get match timeline for a match_id
    // Construct the API URL with the appropriate path parameters
    const apiUrl = `https://4i6m73advi.execute-api.us-west-1.amazonaws.com/rgapi/match_timeline/${region}/${match_id}`;

    try {
      let response = await fetch(apiUrl, {
        headers: {
          'Content-Type': 'application/json',
        },
        mode: 'cors'
      });
        
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      let json = await response.json();
      return json.info;
    } catch (error) {
      console.error("Fetch error: ", error.message);
      // handle the error
    }
  }

  /*
    HELPER FUNCTIONS
    -----------------

    The below functions all assist in the data cleaning process
  */

  function get_participant(participants, puuid){//Function to get the id of a participant given their puuid
    let id = -1;
    participants.forEach(element => {
      if(element['puuid'] == puuid){
        id = element['participantId'];
      }
    })
    return id;
  }

  function champ_map_and_role(match_detail, puuid){//Map puuids in a match to champions played. For certain champions, change the names. Also return player's role
    let champ_names = {};
    let role = "";
    match_detail['participants'].forEach(player =>{
      let champ = player['championName'];
      if(Object.keys(to_change).includes(champ)){
        champ_names[player['puuid']] = to_change[champ]
      }
      else{
        champ_names[player['puuid']] = player['championName']
      }
      if (player['puuid'] == puuid){
        role = player['teamPosition'];
      }
    });
    return [champ_names, role];
  }

  function get_role_opponent(match_detail, puuid, role){//Get puuid of role opponent
    let opp = "";
    match_detail['participants'].forEach(player =>{
      if(role == player['teamPosition'] && puuid != player['puuid']){
        opp = player['puuid'];
      }
    });
    return opp;
  }

  function getRandomColor() {
    var letters = '0123456789ABCDEF';
    var color = '#';
    for (var i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  /*
    DATA CLEANING FUNCTIONS 
    -------------------------

    The below functions all carry out the data cleaning process
  */


  function get_match_gold(match_detail, match_timeline, puuid){ //Gets the gold data (player's and opponent's) for one match given a puuid
    //Get the participant ID of the given person
    let participant = get_participant(match_timeline['participants'], puuid);

    //Map each player's puuid in the match to the champion they were playing
    let [champ_names, role] = champ_map_and_role(match_detail, puuid);

    let opponent = get_role_opponent(match_detail, puuid, role);
    let opponent_participant = get_participant(match_timeline['participants'], opponent);

    //Create the data for the player's gold over time for this game
    let times = {};
    let opponent_times = {};
    match_timeline['frames'].forEach(frame => {
      times[[champ_names[puuid], Math.round(frame['timestamp'] / 60000)]] = [frame['participantFrames'][`${participant}`]['totalGold']];
      opponent_times[[champ_names[opponent], Math.round(frame['timestamp'] / 60000)]] = [frame['participantFrames'][`${opponent_participant}`]['totalGold']];
    })
    return [times, opponent_times];
  }


  function get_all_gold_data(match_ids, match_details, match_timelines, puuid){//Get 
    let gold_over_time = {};
    let opp_gold_over_time = {};

    for (let match_id of match_ids) {
      let [match_times, opponent_times] = get_match_gold(match_details[match_id], match_timelines[match_id], puuid);

      //For each key (such as [Aatrox, 5]) append the value of the gold if it exists and create a new key for it if not.
      for (let key in match_times) {
        if (gold_over_time.hasOwnProperty(key)) { 
          gold_over_time[key] = gold_over_time[key].concat(match_times[key]); 
        } else { 
          gold_over_time[key] = match_times[key]; 
        }
      }

      //Do the same for opponents
      for (let key in opponent_times) {
        if (opp_gold_over_time.hasOwnProperty(key)) { 
          opp_gold_over_time[key] = opp_gold_over_time[key].concat(opponent_times[key]); 
        } else { 
          opp_gold_over_time[key] = opponent_times[key]; 
        }
      }
    }
    return [gold_over_time, opp_gold_over_time];
  }

  function get_all_champ_data(match_ids, match_details, puuid){
    let champs_played = {};

    for (let match_id of match_ids) {
      let champ_names = champ_map_and_role(match_details[match_id])[0];
      let champ = champ_names[puuid];

      if (champs_played.hasOwnProperty(champ)) { 
          champs_played[champ]++; 
      } 
      else{ 
          champs_played[champ] = 1; 
      }
        
    }
    return champs_played;
  }

  function reformat_gold_data(data){//Reformat the data we've obtained so it's the same format as from Project 3
    let clean_data = [];
    const average = array => array.reduce((a, b) => a + b) / array.length;
    for(let element in data){
      let index = element.split(',');
      let this_element = {};
      this_element['time'] = parseFloat(index[1]);
      this_element['champion'] = index[0];
      this_element['gold'] = average(data[element]);
      clean_data.push(this_element);
    }
    return clean_data;
  }
  
  function get_overall_gold(reformatted_data){
    let map = d3.group(reformatted_data, d => d.time);
    const average = array => array.reduce((a, b) => a + b) / array.length;
    let overall_data = [];
    map.forEach(element => {
      let this_element = {};
      let times = element.map(function (el) { return el.gold; });
      this_element['time'] = element[0]['time'];
      this_element['champion'] = 'Your Average';
      this_element['gold'] = average(times);
      overall_data.push(this_element);
    });
    return overall_data;
  }

  async function tally_data() {

    loading = true;
    
    
    const puuid = await get_puuid_by_name_tag(region, gameName, tagLine);
    const match_ids = await get_most_recent_match_ids_by_puuid(region, puuid, NUM_MATCHES);
    const match_details = {};
    const match_timelines = {};
    for (let match_id of match_ids) {
     match_details[match_id] = await get_match_detail_by_match_id(region, match_id);
     match_timelines[match_id] = await get_match_timeline_by_match_id(region, match_id);
    }

    /*
      Add new data processing function here
    */

    //get champ data for bar graph
    const champs_played_data = get_all_champ_data(match_ids, match_details, puuid);
    
    

    // get gold data for line graph
    const [gold_over_time_data, opp_gold_data] = get_all_gold_data(match_ids, match_details, match_timelines, puuid);

    

    const clean_gold_data = reformat_gold_data(gold_over_time_data);
    const overall_data = get_overall_gold(clean_gold_data);
    const clean_opp_gold_data = reformat_gold_data(opp_gold_data);

    const opponent_champs = Array.from((d3.group(clean_opp_gold_data, d => d.champion)).keys());
    const all_champs = Object.values(champs_played_data).concat(opponent_champs).concat(['Your Average'])

    let random_colors = [];

    all_champs.forEach(champ =>{
      let color = getRandomColor();
      random_colors.push(color);
    });

    const color_scheme = d3.scaleOrdinal().domain(all_champs).range(random_colors);


    dispatch('sending_data', {gold_data: clean_gold_data, champ_data: champs_played_data, opponent_data: clean_opp_gold_data.concat(overall_data), color: color_scheme});
    loading = false;
  }


</script>

<main>
  <div id="searchbar">
    <div class="region-dropdown-wrapper">
        <Dropdown theme="dark">
          <DropdownToggle color="dark" caret>
            {region===undefined ? "Region" : region.charAt(0).toUpperCase()+region.slice(1)}
          </DropdownToggle>
          <DropdownMenu end>
            {#each validRegions as reg}
              <DropdownItem on:click={() => region = reg}>
                {reg.charAt(0).toUpperCase()+reg.slice(1)}
              </DropdownItem>
            {/each}
          </DropdownMenu>
        </Dropdown>
    </div>
    
    <div class="input-boxes-wrapper">
      <Input id="gamename" theme="dark" placeholder="Game Name" maxlength="16" bind:value={gameName}/>
      <InputGroup theme="dark" id="inputgroup">
        <InputGroupText>#</InputGroupText>
        <Input id="tag" placeholder="Tag" maxlength="5" bind:value={tagLine}/>
      </InputGroup>
    </div>

      <button id="searchbutton" type="button" class="btn btn-primary" disabled={loading} on:click={()=>{
        //Setting the account to Faker's just for checkpoint purposes, can be erased later

        tally_data();
      }}>Search</button>

  </div>
  
  <div id = "spinner">
    {#if loading}
      <Spinner type="border" color="primary" />
    {/if}
  </div>
  
</main>

<style>
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300..800;1,300..800&display=swap');


:root {
  --grey-bg: #31313C;
}

#searchbar {
  display: flex;
  flex-direction: row;
  align-items: center;
  background-color: var(--grey-bg);
  border-radius: 50px;
  padding: 10px 20px;
  gap: 10px;
  position: relative;
  top: 30%;
}

.input-boxes-wrapper {
  display: flex;
  flex-direction: row;
  align-items: center;
}

:global(#gamename) {
  width: 200px;
}

:global(#inputgroup) {
  display: flex;
  flex-direction: row;
  white-space: nowrap;
  width: 115px;
}

#spinner {
  position: relative;
  top: 35%;
}

</style>