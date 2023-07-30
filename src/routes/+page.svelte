<script>
// @ts-nocheck

	import { onMount } from 'svelte';

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
	
	let timer = null;
	let clock = null;
	let now = new Date();
	let timings = [];
	let delta = 0;
	let run = 1;

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
		}
	}

	$: {
		if (delta < 0) {
			delta = 0;
		}
	}

	$: {
		console.log(timings);
		console.log(run);
	}
</script>

<div class="content">
	<div class="column">
		<h1>Time: {delta.toFixed(1)}</h1>
		<button disabled={timer !== null && ["climbing", "lowering"].includes(timer.type)} on:click={() => startTimer("climbing")}>Climbing</button>
		<button disabled={timer === null || ["take", "lowering", "resting"].includes(timer.type)} on:click={() => startTimer("take")}>Take</button>
		<button disabled={timer === null || ["lowering", "resting"].includes(timer.type)} on:click={() => startTimer("lowering")}>Lowering</button>
		<button disabled={timer === null || ["resting", "take", "climbing"].includes(timer.type)} on:click={() => startTimer("resting")}>Resting</button>
		<button disabled={timer === null || ["climbing", "take"].includes(timer.type)} on:click={() => { clearTimer(); run++;} }>Stop</button>

		{#if timings.length > 0}
			<table>
				<tr>
					<th>Run #</th>
					<th>Type</th>
					<th>Time (s)</th>
				</tr>
			{#each timings as timer}
				<tr>
					<td>{timer.run}</td>
					<td>{timer.type}</td>
					<td>{timer.totalTime()}</td>
				</tr>
			{/each}
			</table>
		{/if}

		{#if run > 1}
			<h1>Summary</h1>
			<table>
				<tr>
					<th>Run</th>
					<th>Time on the wall</th>
					<th>% Climbing</th>
					<th>% Resting</th>
					<th>Total run time</th>
				</tr>

				{#each new Set(timings.map(t => t.run)) as runNumber}
					{@const wallTime = timings.
							 filter(t => t.run === runNumber && ["climbing", "take"].includes(t.type)).
							 reduce((old, next) => old + next.totalTime(), 0)}
					<tr>
						<td>{runNumber}</td>
						<td>{wallTime.toFixed(1)}</td>
						<td>{(timings.
							filter(t => t.run === runNumber && t.type === "climbing").
							reduce((old, next) => old + next.totalTime(), 0) / wallTime * 100).toFixed()}</td>
						<td>{(timings.
							filter(t => t.run === runNumber && t.type === "take").
							reduce((old, next) => old + next.totalTime(), 0) / wallTime * 100).toFixed()}</td>
						<td>{(timings.filter(r => r.run === runNumber).reduce((old, next) => old + next.totalTime(), 0)).toFixed(1)}s</td>
					</tr>
				{/each}
			</table>
		{/if}
			
	</div>
</div>

<style>
	.content {
		display: flex;
		justify-content: center;
		height: 100%;
		width: 100%;
		background-color: white;
		color: black;
		overflow-y: scroll
	}
	
	.column {
		display: flex;
		flex-direction: column;
		gap: 0.5em;
	}

	table, th, td {
		border: 1px solid;
	}
</style>