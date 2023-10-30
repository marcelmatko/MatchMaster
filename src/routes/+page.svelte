<script lang="ts">
	import { emoji } from './emoji'

  type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

  // game state
  let state: State = 'start'
  // size of the grid
  let size = 20
  // game grid
  let grid = createGrid()
  // used to check if game is over
  let maxMatches = grid.length / 2
  // selected cards
  let selected: number[] = []
  // matched cards
  let matches: string[] = []
  // timer
  let timerId: number | null = null
  let time = 50

 function startGameTimer() {
  function countDown() {
    state != 'paused' && (time -= 1)
  }
  timerId = setInterval(countDown, 1000)
 }

 function createGrid() {
		// only want unique cards
		let cards = new Set<string>()
		// half because we duplicate the cards
		let maxSize = size / 2

		while (cards.size < maxSize) {
			// pick random emoji
			const randomIndex = Math.floor(Math.random() * emoji.length)
			cards.add(emoji[randomIndex])
		}

		// duplicate and shuffle cards
		return shuffle([...cards, ...cards])
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5)
	}
  
  function selectCard(cardIndex: number) {
    selected = selected.concat(cardIndex)
  }

  function matchCards() {
    const [first, second] = selected

    if(grid[first] == grid[second]){
      matches = matches.concat(grid[first])
    }

    setTimeout(() => (selected = []), 300)
  }

  function resetGame() {
    state = 'playing'
    timerId && clearInterval(timerId)
    grid = createGrid()
    maxMatches = grid.length / 2
    selected = []
    matches = []
    timerId = null
    time = 50
  }

  function gameWon() {
    state = 'won'
  }

  function gameLost() {
    state = 'lost'
  }

  $: if (state == 'playing' && !timerId && selected.length == 1) {
    startGameTimer()
  }

  $: selected.length == 2 && matchCards()
  $: maxMatches == matches.length && gameWon()
  $: time == 0 && gameLost()

  $: console.log({state, selected, matches})

</script>

<!--
{#if state == 'start'}
<h1>Flip n' Match</h1>
<button on:click={() => state = 'playing'}>Start</button>
{/if}
-->

{#if state == 'start'}
<h1 class="title">Flip n' Match</h1>
{/if}
{#if state == 'playing'}
<h1 class="title">Flip n' Match</h1>
<h1 class="timer" class:pulse={time < 10}>{time}</h1>
{/if}
{#if state == 'won'}
<h1 class="title">Flip n' Match</h1>
<h1>You win!</h1>
{/if}
{#if state == 'lost'}
<h1 class="title">Flip n' Match</h1>
<h1>You lost!</h1>
{/if}

<div class="matches">
  {#each matches as card}
  <div>{card}</div>
  {/each}
</div>
<div class="cards">
  {#each grid as card, cardIndex}
    {@const isSelected = selected.includes(cardIndex)}
    {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
    {@const match = matches.includes(card)}

    <button
    class="card"
    class:disabled={state == 'start' || state == 'lost' || state == 'won'}
    class:selected={isSelected}
    class:flip={isSelectedOrMatched}
    disabled={isSelectedOrMatched} 
    on:click={() => selectCard(cardIndex)}
    >
      <div class="back" class:match>{card}</div>
    </button>
  {/each}
</div>
<div style="display: flex; justify-content: center; padding: 3rem;">
  {#if state == 'start'}
  <button class="btn" on:click={() => state = 'playing'}>Start</button>
  {/if}
  {#if state == 'won' || state == 'lost'}
  <button class="btn" on:click={resetGame} >Play again</button>
  {/if}
</div>


<style>
  .btn {

    border: 4px solid orange;
  }
  .title {
    font-size: 108px;
    font-family: 'Nabla', cursive;
  }
  .cards {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 0.4rem;;
  }
  .card {
    height: 140px;
    width: 140px;
    font-size: 4rem;
    background-color: var(--bg-2);
    transition: rotate 0.3s ease-out;
    transform-style: preserve-3d;

    &.selected {
      border: 4px solid var(--border);
    }

    &.flip {
      rotate: y 180deg;
      pointer-events: none;
    }

    &.disabled {
      pointer-events: none;
    }

    & .back {
      position: absolute;
      inset: 0;
      display: grid;
      place-content: center;
      backface-visibility: hidden;
      rotate: y 180deg;
    }

    & .match  {
      transition: opacity 0.3 ease-out;
      opacity: 0.4;
    }
  }

  .card:hover {
    background-color: #2e3545;
  }

  .matches {
    display: flex;
    gap: 1rem;
    margin-block: 2rem;
    font-size: 3rem;
  }

  .timer {
    transition: color 0.3s ease;
  }

  .pulse {
    color: var(--pulse);
    animation: pulse 1s infinite ease;
  }

  @keyframes pulse {
    to {
      scale: 1.4;
    }
  }

</style>