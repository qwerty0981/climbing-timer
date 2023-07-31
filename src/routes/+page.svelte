<script>
// @ts-nocheck

	import { onMount } from 'svelte';
	import Card, { Content } from '@smui/card'
  import Button from "@smui/button";

	class Timer {
		_start = null;
		_end = null;
		type;
		run;
		
		/**
         * @param {string} type
         * @param {number} run
         */
		constructor(type, run) {
			this.type = type;
			this.run = run;
		}

		start() {
			this._start = new Date();
		}

		stop() {
			this._end = new Date();
		}

		elapsedTime(now) {
			if (this._start !== null) {
				return calcDelta(this._start, now)
			}
			return 0;
		}

		totalTime() {
			return calcDelta(this._start, this._end);
		}
	}
	
  let state = "stop";

	let timer = null;
	let clock = null;
	let now = new Date();
	let timings = [];
	let delta = 0;
	let run = 1;

  function stateChange(nextState) {
    if (nextState === "stop") {
      clearTimer()
      if (state !== "resting") {
        run++;
      }
    } else {
      startTimer(nextState);
    }
    state = nextState;
  }

	function startTimer(type) {
		if (timer !== null) {
			clearTimer()
		}

		timer = new Timer(type, run);
		timer.start();
		clock = setInterval(() => {
			now = new Date()
			console.log(timer);
		}, 100);
		if (type === "resting") {
			run++;
		}
	}

	function clearTimer() {
		clearInterval(clock);
		timer.stop();
		timings = [...timings, timer];
		timer = null;
		delta = 0;
	}

	function calcDelta(start, end) {
		return (end.getTime() - start.getTime()) / 1000;
	}

	onMount(() => {
		return () => {
			clearInterval(clock);
		}
	})


	$: {
		if (timer !== null) {
			delta = timer.elapsedTime(now);
      if (delta < 0) {
        delta = 0;
      }
		}
	}
</script>

<Card padded variant="raised" style="height: 100%">
  <div class="timer">
    <h1>Time: {delta.toFixed(1)}</h1>
    {#if state === "stop"}
      <Button class="user-button" style="background-color: green" variant="raised" on:click={() => stateChange("climbing")}>Climbing</Button>
    {:else if state === "climbing"}
      <Button class="user-button" style="background-color: orange" variant="raised" on:click={() => stateChange("take")}>Take</Button>
      <Button class="user-button" style="background-color: red" variant="raised" on:click={() => stateChange("lowering")}>Lowering</Button>
    {:else if state === "take"}
      <Button class="user-button" style="background-color: green" variant="raised" on:click={() => stateChange("climbing")}>Climbing</Button>
      <Button class="user-button" style="background-color: red" variant="raised" on:click={() => stateChange("lowering")}>Lowering</Button>
    {:else if state === "lowering"}
      <Button class="user-button" style="background-color: blue" variant="raised" on:click={() => stateChange("resting")}>Resting</Button>
      <Button class="user-button" style="background-color: red" variant="raised" on:click={() => stateChange("stop")}>Stop</Button>
    {:else if state === "resting"}
      <Button class="user-button" style="background-color: green" variant="raised" on:click={() => stateChange("climbing")}>Climbing</Button>
      <Button class="user-button" style="background-color: red" variant="raised" on:click={() => stateChange("stop")}>Stop</Button>
    {/if}
    </div>
    {#if state === "stop"}
    <div style="flex-grow: 1">
      <h1>Summary</h1>
      <table style="width: 100%">
        <tr>
          <th>#</th>
          <th>Climbing</th>
          <th>Resting</th>
          <th>ToW</th>
          <th>Time Between</th>
        </tr>

        {#each new Set(timings.map(t => t.run)) as runNumber}
          {@const wallTime = timings.
               filter(t => t.run === runNumber && ["climbing", "take"].includes(t.type)).
               reduce((old, next) => old + next.totalTime(), 0)}
          <tr>
            <td>{runNumber}</td>
            <td>{timings.
              filter(t => t.run === runNumber && t.type === "climbing").
              reduce((old, next) => old + next.totalTime(), 0).toFixed(1)}s</td>
            <td>{timings.
              filter(t => t.run === runNumber && t.type === "take").
              reduce((old, next) => old + next.totalTime(), 0).toFixed(1)}s</td>
            <td>{wallTime.toFixed(1)}s</td>
            <td>{(timings.filter(r => r.run === runNumber && ["lowering", "resting"].includes(r.type)).reduce((old, next) => old + next.totalTime(), 0)).toFixed(1)}s</td>
          </tr>
        {/each}
      </table>
    </div>
    {/if}
</Card>

<style>
	/* .content {
		display: flex;
		justify-content: center;
		height: 100%;
		width: 100%;
		background-color: white;
		color: black;
		overflow-y: scroll
	} */
  
  .timer {
    width: 100%;
    flex-grow: 1;
    display: flex;
    align-items: center;
    flex-direction: column;
    gap: 0.5em;
  }

  * :global(.user-button) {
    width: 100%;
    flex-grow: 1;
  }

	table, th, td {
		border: 1px solid;
    text-align: center;
	}

</style>