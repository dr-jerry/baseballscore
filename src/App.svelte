<script>
	// Inspired by https://svelte.dev/repl/810b0f1e16ac4bbd8af8ba25d5e0deff?version=3.4.2.
	import {flip} from 'svelte/animate';
	
	let theGame = {"thuis": {innings: [{points: 0, outs: 0}],"border-color": "orange", "bg-color": "blue", "text-color": "white", teamname: "thamen hbu12"
	, names:["Arno", "Finn", "Sasha", "Djaeno", "Steffan", "Jayjay", "Pascal"], loc: "field"},
	   "uit": {innings: [{points: 0, outs: 0, bats: ""}], "border-color": "red", "bg-color": "yellow", "text-color": "black", teamname: "ovvo-survivors hbu2"
	   , names:["Jack", "Frans", "Arie", "Dessel", "Nour", "Izzy", "Ben"], loc: "betting"}}

    theGame.thuis.players = theGame.thuis.names.map((n,i) => ({name: n, innings: [{bats: "", loc: "bench"}], loc: "field", index: i, teamname: theGame.thuis.teamname}))
	theGame.uit.players = theGame.uit.names.map((n,i) => ({name: n, innings: [{bats: "", loc: "bench"}], loc: "bench", index: i, teamname: theGame.uit.teamname}))

	let batting = "uit";
	let round = 1;

	function doBat(char) {
		let index = bases["home"][0].index
		let bat = theGame[batting].players[index].innings.slice(-1)[0].bats
		theGame = modifyPlayer(batting, index, {inning: {bats: `${bat}${char}`}})
	}

	function baseOccupation(teams, teamid) {
		const locs = ["bench", "home", "one", "two", "three"]
		return locs.reduce((map, loc) => {map[loc] = findPlayers(loc, teams, teamid);
					return map}, {})
	}

	function findPlayers(loc, teams, teamid) {
		console.log("find players " + loc);
		return teams[teamid].players ? teams[teamid].players.filter(p => p.loc === loc) : []
	}

	$: bases = baseOccupation(theGame, batting)
	$: console.log("bases" + JSON.stringify(bases));

	function modifyPlayer(ind, index, value) {
		let players = theGame[ind].players
		let thePlayer = players[index]
		const {inning, ...rest} = value
		let thisInning = {...thePlayer.innings.slice(-1)[0], ...inning}
		return {...theGame, ...{[ind]: {...theGame[ind], 
			...{players: [...players.slice(0,index), 
				{...thePlayer, ...rest, ...{innings : [...thePlayer.innings.slice(0,-1), thisInning]}}
					, ...players.slice(index + 1)]}}}}
	}

	function dragStart(event, source, player) {
		//event.preventDefault();
		let data = JSON.stringify({source, player});
		event.dataTransfer.setData('text/plain', data);
		console.log("set " + data);
	}
	
	function movePlayer(event, target, teamid) {
		event.preventDefault();
		// without above, iPhone and iPad forward the page to a google page with the dataTransfer in the search bar.
		let thePlayer = JSON.parse(event.dataTransfer.getData("text/plain"));
		console.log("target: " + target + " source : " + thePlayer.source + " player: " +  JSON.stringify(thePlayer) + " hello ");
		//console.log("honks " + JSON.stringify(honks));
		if (thePlayer.source === "bench" && target === "home") { 
			theGame = modifyPlayer(batting, thePlayer.player.index, {loc: "home"})
		} else if (target === "one" && thePlayer.source === "home") {
			theGame = modifyPlayer(batting, thePlayer.player.index, {loc: "one"})
		} else if (target === "two" && thePlayer.source === "one") {
			theGame = modifyPlayer(batting, thePlayer.player.index, {loc: "two"})
		} else if (target === "three" && thePlayer.source === "two") {
			theGame = modifyPlayer(batting, thePlayer.player.index, {loc: "three"})
		} else if (target === "home" && thePlayer.source === "three") {
			theGame = modifyPlayer(batting, thePlayer.player.index, {loc: "bench"})
			theGame = modifyPlayer(batting, thePlayer.player.index, {inning: {nr: round, score: "1"}})
		} else if (target === "bench") {
			honks = honks.map(h => h.filter(p => p.name !== thePlayer.player.name));
			
			// [[...honks[0].filtet(p => p !== player)],[...honks[1].filtet(p => p !== player)]
			// 	,[...honks[2].filtet(p => p !== player)],[...honks[3].filtet(p => p !== player)]];
			bench = [...bench, {...thePlayer.player, scores: [...thePlayer.player.scores,"0"]}]
		}
		console.log("honks is " + JSON.stringify(theGame))

	}

</script>
{#if bases["home"].length > 0}
<div><div class ="button" on:click={e => doBat("S")}>Strike</div><div class="button" on:click={e => doBat("B")}>Ball</div></div>
{/if}
<p>Drag Your players on the bases.</p>
<div class="bench" on:dragenter={(event) => {event.preventDefault();console.log("enter bench" + event.dataTransfer.getData("text/plain"))}}
	on:dragleave={() => console.log("leave bench")}
	on:drop={event => {console.log("drop with: " + JSON.stringify(event));movePlayer(event, "bench")}}
	ondragover="return false">
  {#each bases["bench"] as player, nameIndex(player)}
    <div class="player" draggable={nameIndex == 0} on:dragstart={event => dragStart(event, "bench", player)}>
      {player.name}
	</div>
  {/each}
</div>
<div class="field">
	<div class="row">
		<div class="honk left" id="home" on:dragenter={(event) => {event.preventDefault();console.log("enter bench" + event.dataTransfer.getData("text/plain"))}}
			on:dragleave={() => console.log("leave home")}
			on:drop={event => {console.log("drop with: " + JSON.stringify(event));movePlayer(event, "home")}}
			ondragover="return false">
		home
		   {#each bases["home"] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "home", player)}>{player.name}</div>
		   {/each}
		</div>
		<div class="honk right" id="three" on:dragenter={(event) => {event.preventDefault()}}
			on:dragleave={() => console.log("leave three")}
			on:drop={event => movePlayer(event, "three")}
			ondragover="return false">
		3
		   {#each bases["three"] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "three", player)}>{player.name}</div>
		   {/each}
		</div>
	</div>
	<div class="row">
		<div class="honk left" id="one" on:dragenter={(event) => {event.preventDefault()}}
			on:dragleave={() => console.log("leave one")}
			on:drop={event => movePlayer(event, "one")}
			ondragover="return false">
		1
		   {#each bases["one"] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "one", player)}>{player.name}</div>
		   {/each}
		</div>
		<div class="honk right" id="two" on:dragenter={(event) => {event.preventDefault()}}
			on:dragleave={() => console.log("leave two")}
			on:drop={event => movePlayer(event, "two")}
			ondragover="return false">
		2
		   {#each bases["two"] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "two", player)}>{player.name}</div>
		   {/each}
		</div>
	</div>
</div>

<style>
	.field {
		width: 68%;
		background-color: green;
		float: right;
		display: block;
	}
	.row {
		display: block;
		width: 100%;
	}
 
  .bench {
     width: 30%;
     float: left;
     margin-left: 0.2em;
	 padding-left: 0.4em;
  }
  .honk {
	  margin: 1em;
	  border: 0.2em;
	  background-color: black;
	  width: 2.5em;
	  height: 4em;
	  color: white;
	  padding: 0.8em;
  }
  .honk.left {
	  float: left;
	  clear:right;
  }
  .honk.right {
	  float: right;
  }

  .player {
     display: inline-block;
     background-color: lightgray;
     cursor: pointer;
     width:80%;
	 margin: 2px;
	 padding: 1px;
	 padding-left:0.7em;
  }
  .player:hover {
	  background-color: blueviolet;
	  color: red;
  }
    
	.hovering {
		border-color: orange;
	}
	.item {
		display: inline; /* required for flip to work */
	}
	li {
		background-color: lightgray;
		cursor: pointer;
		display: inline-block;
		margin-right: 10px;
		padding: 10px;
	}
	li:hover {
		background: orange;
		color: white;
	}
  ul {
		border: solid lightgray 1px;
		display: flex; /* required for drag & drop to work when .item display is inline */
		height: 40px; /* needed when empty */
		padding: 10px;
	}
</style>