<script>
	// Inspired by https://svelte.dev/repl/810b0f1e16ac4bbd8af8ba25d5e0deff?version=3.4.2.
	import {flip} from 'svelte/animate';
	
	let names = ["Arno", "Finn", "Sasha", "Djaeno", "Stefan"];

	let bench = names.map((n,i) => ({name: n, scores: [], index: i}))
	let honks = [[],[],[],[]];

	function dragStart(event, source, player) {
		//event.preventDefault();
		let data = JSON.stringify({source, player});
		event.dataTransfer.setData('text/plain', data);
		console.log("set " + data);
	}
	
	function movePlayer(event, target) {
		event.preventDefault();
		// without above, iPhone and iPad forward the page to a google page with the dataTransfer in the search bar.
		let thePlayer = JSON.parse(event.dataTransfer.getData("text/plain"));
		console.log("target: " + target + " source : " + thePlayer.source + " player: " +  JSON.stringify(thePlayer) + " hello ");
		console.log("honks " + JSON.stringify(honks));
		if (thePlayer.source === "bench" && target === "home") { 
			console.log(JSON.stringify(thePlayer))
			bench = bench.filter((player, i) => i !== 0)
			honks = [[...honks[0],thePlayer.player], ...honks.slice(1,4)];
		} else if (target === "one" && thePlayer.source === "home") {
			honks = [[], [...honks[1],thePlayer.player], ...honks.slice(2,4)];
		} else if (target === "two" && thePlayer.source === "one") {
			honks = [[...honks[0]], [...honks[1].slice(1, honks[1].length)], [...honks[2],thePlayer.player], [...honks[3]]];
		} else if (target === "three" && thePlayer.source === "two") {
			honks = [[...honks[0]], [...honks[1]],[...honks[2].slice(1, honks[2].length)], [...honks[3],thePlayer.player]];
		} else if (target === "home" && thePlayer.source === "three") {
			honks = [...honks.slice(0,3),[...honks[3].slice(1,honks[3].length)]];
			bench = [...bench, {...thePlayer.player, scores: [...thePlayer.player.scores,"1"]}]
		} else if (target === "bench") {
			honks = honks.map(h => h.filter(p => p.name !== thePlayer.player.name));
			
			// [[...honks[0].filtet(p => p !== player)],[...honks[1].filtet(p => p !== player)]
			// 	,[...honks[2].filtet(p => p !== player)],[...honks[3].filtet(p => p !== player)]];
			bench = [...bench, {...thePlayer.player, scores: [...thePlayer.player.scores,"0"]}]
		}
		console.log("honks is " + JSON.stringify(honks))

	}

</script>

<p>Drag Your players on the bases.</p>
<div class="bench" on:dragenter={(event) => {event.preventDefault();console.log("enter bench" + event.dataTransfer.getData("text/plain"))}}
	on:dragleave={() => console.log("leave bench")}
	on:drop={event => {console.log("drop with: " + JSON.stringify(event));movePlayer(event, "bench")}}
	ondragover="return false">
  {#each bench as player, nameIndex(player) }
    <div class="player" draggable={nameIndex == 0} on:dragstart={event => dragStart(event, "bench", player)}>
      {player.name + " : " + player.scores.join(',')}
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
		{#if honks[0] !== undefined && honks[0].length>0}
		   {#each honks[0] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "home", player)}>{player.name}</div>
		   {/each}
		{/if}
		</div>
		<div class="honk right" id="three" on:dragenter={(event) => {event.preventDefault()}}
			on:dragleave={() => console.log("leave three")}
			on:drop={event => movePlayer(event, "three")}
			ondragover="return false">
		3
		{#if honks[3] !== undefined && honks[3].length>0}
		   {#each honks[3] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "three", player)}>{player.name}</div>
		   {/each}
		{/if}
		</div>
	</div>
	<div class="row">
		<div class="honk left" id="one" on:dragenter={(event) => {event.preventDefault()}}
			on:dragleave={() => console.log("leave one")}
			on:drop={event => movePlayer(event, "one")}
			ondragover="return false">
		1
		{#if honks[1] !== undefined && honks[1].length>0}
		   {#each honks[1] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "one", player)}>{player.name}</div>
		   {/each}
		{/if}
		</div>
		<div class="honk right" id="two" on:dragenter={(event) => {event.preventDefault()}}
			on:dragleave={() => console.log("leave two")}
			on:drop={event => movePlayer(event, "two")}
			ondragover="return false">
		2
		{#if honks[2] !== undefined && honks[2].length>0}
		   {#each honks[2] as player, i(player)}
		   <div clas="player honk" draggable={i == 0} on:dragstart={event => dragStart(event, "two", player)}>{player.name}</div>
		   {/each}
		{/if}
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