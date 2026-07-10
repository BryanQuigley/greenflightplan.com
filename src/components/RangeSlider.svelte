<script>
	import { range } from '../lib/stores.js';
	import { scalePow } from 'd3';

	const scale = scalePow()
		.domain([0, 1])
		.range([100, 2500])
		.exponent(2.5)
		.clamp(true);

	let track = $state(null);
	let handle = $state(null);
	let ready = $state(false);
	let dragging = $state(false);

	$effect(() => {
		if (track && handle) {
			positionHandle();
			ready = true;
		}
	});

	function positionHandle() {
		if (!track || !handle) return;
		const x = scale.invert($range) * (track.clientWidth - handle.clientWidth) + handle.clientWidth / 2;
		handle.style.left = x + 'px';
	}

	function onPointerDown(e) {
		dragging = true;
		e.target.setPointerCapture(e.pointerId);
	}

	function onPointerMove(e) {
		if (!dragging || !track || !handle) return;
		const rect = track.getBoundingClientRect();
		const offset = e.clientX - rect.left - handle.clientWidth / 2;
		const maxOffset = track.clientWidth - handle.clientWidth;
		const clamped = Math.max(0, Math.min(maxOffset, offset));
		const percent = clamped / maxOffset;
		range.set(Math.round(scale(percent)));
	}

	function onPointerUp() {
		dragging = false;
	}

	function setRange(target) {
		range.set(target);
		positionHandle();
	}

	$effect(() => {
		// Update handle position when range changes externally (e.g. tick click)
		const _ = $range;
		if (track && handle && !dragging) {
			positionHandle();
		}
	});

	let ticks = $state([
		{ range: 100, label: "100" },
		{ range: 124, label: "HA ES-30", class: 'plane' },
		{ range: 460, label: "Wright Spirit", class: 'plane' },
		{ range: 500, label: "500" },
		{ range: 1000, label: "1000" },
		{ range: 1500, label: "1500" },
		{ range: 2000, label: "2000" },
		{ range: 2500, label: "2500" },
	]);

	ticks = ticks.map(tick => ({
		...tick,
		x: scale.invert(tick.range) * 100,
	}));
</script>

<svelte:window onresize={() => positionHandle()} />

<div>
	<div id="title-row">
		<h3>
			Range in miles
			<svg width="11" height="9" viewBox="0 0 11 9" fill="none">
				<path d="M9.89062 5.03906L10.4297 4.5L9.89062 3.98438L6.51562 0.609375L6 0.0703125L4.92188 1.125L5.46094 1.66406L7.54688 3.75H0.75H0V5.25H0.75H7.54688L5.46094 7.35938L4.92188 7.875L6 8.95312L6.51562 8.41406L9.89062 5.03906Z" fill="black"/>
			</svg>
		</h3>
	</div>

	<div id="track" bind:this={track}>
		<div id="bar" class:visible={ready} style="width: {scale.invert(Math.max(102,$range))*100}%"></div>
		<div id="handle"
			bind:this={handle}
			class:visible={ready}
			class:dragging
			role="slider"
			aria-valuemin="100"
			aria-valuemax="2500"
			aria-valuenow={$range}
			tabindex="0"
			onpointerdown={onPointerDown}
			onpointermove={onPointerMove}
			onpointerup={onPointerUp}
			data-call-to-action="Drag me!"
		>
			<svg width="7" height="10" viewBox="0 0 7 10" fill="none">
				<path d="M6.06505 1.28995C6.06505 1.47135 6.00459 1.63259 5.88365 1.75353L2.47734 5.15984L5.88365 8.58631C6.14568 8.82818 6.14568 9.25145 5.88365 9.49332C5.64178 9.75534 5.21851 9.75534 4.97664 9.49332L1.10675 5.62342C0.844726 5.38156 0.844726 4.95829 1.10675 4.71642L4.97665 0.846522C5.21851 0.584498 5.64178 0.584498 5.88365 0.846523C6.00459 0.967457 6.06505 1.1287 6.06505 1.28995Z" fill="var(--neutral-200)"/>
			</svg>
			<span>{$range}</span>
			<svg width="6" height="10" viewBox="0 0 6 10" fill="none">
				<path d="M0.583385 9.02975C0.583385 8.84835 0.643852 8.68711 0.764786 8.56617L4.1711 5.15986L0.764786 1.73339C0.502762 1.49152 0.502762 1.06825 0.764786 0.826383C1.00665 0.564359 1.42992 0.564359 1.67179 0.826383L5.54169 4.69628C5.80371 4.93815 5.80371 5.36142 5.54169 5.60329L1.67179 9.47318C1.42992 9.7352 1.00665 9.7352 0.764786 9.47318C0.643852 9.35225 0.583385 9.191 0.583385 9.02975Z" fill="var(--neutral-200)"/>
			</svg>
		</div>

		<div id="ticks">
		{#each ticks as tick}
			<!-- svelte-ignore a11y_click_events_have_key_events -->
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<span
				style="margin-left: {tick.x}%;"
				class="tick"
				class:plane="{tick.class==='plane'}"
				onclick={() => setRange(tick.range)}
			>
				{tick.label}
			</span>
			{/each}
		</div>
	</div>
</div>

<style>
	@keyframes pulse {
		50% { color: var(--neutral-700); transform: scale(1.05); }
	}

	#title-row {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
	}
	#title-row h3 {
		font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
		font-size: 14px;
		font-weight: 600;
		line-height: 17px;
		letter-spacing: 0em;
		color: var(--neutral-700);
	}
	#title-row path {
		fill: var(--neutral-700);
	}

	#track {
		width: 100%;
		background-color: var(--neutral-200);
		height: 4px;
		position: relative;
		margin: 2.25rem 0 1.5rem 0;
		padding: 0 42px;
		box-sizing: border-box;
	}

	#track #bar {
		background-color: var(--neutral-500);
		height: 100%;
		position: absolute;
		left: 0;
		transition: opacity .25s ease-in-out;
	}
	#track #bar:not(.visible) {
		opacity: 0;
	}

	#track #handle {
		z-index: 2;
		background-color: #000;
		color: #fff;
		width: 94px;
		height: 36px;
		top: -16px;
		left: -47px;
		border-radius: 20px;
		position: absolute;
		display: flex;
		justify-content: space-between;
		align-items: center;
		box-sizing: border-box;
		padding: 0 8px !important;
		flex-shrink: 0;
		cursor: grab;
		touch-action: none;
		box-shadow: 0px 4px 6px -1px rgba(0, 0, 0, 0.1), 0px 2px 4px -1px rgba(0, 0, 0, 0.06);
		transition: opacity .25s ease-in-out;
		user-select: none;
	}
	#track #handle:not(.visible) {
		opacity: 0;
	}
	#track #handle:hover {
		transform: scale(1.05);
		outline: 3px solid rgba(0,0,0,.15);
	}
	#track #handle.dragging {
		cursor: grabbing;
		transform: scale(1.05);
	}
	#track #handle * {
		pointer-events: none;
	}
	#track #handle span {
		font-size: 16px;
		font-weight: 600;
		line-height: 19px;
		letter-spacing: 0em;
		text-align: center;
	}
	#track #handle:not(.dragging):before {
		content: attr(data-call-to-action);
		position: absolute;
		height: 36px;
		width: 100%;
		pointer-events: none;
		bottom: -100%;
		left: 0;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		font-size: 14px;
		font-weight: 400;
		line-height: 17px;
		letter-spacing: 0em;
		text-align: center;
		color: var(--neutral-500);
		animation: pulse 2s infinite ease-in-out;
	}

	#ticks {
		position: relative;
		width: 100%;
		z-index: 1;
	}
	#ticks .tick {
		position: absolute;
		width: 1px;
		display: flex;
		justify-content: center;
		flex-direction: row;
		font-size: 12px;
		font-weight: 400;
		line-height: 14px;
		letter-spacing: 0em;
		text-align: center;
		color: var(--neutral-500);
		cursor: pointer;
		height: 14px;
		align-items: flex-start;
		white-space: nowrap;
		margin-top: 4px;
		border-top: 4px solid var(--neutral-300);
	}
	#ticks .tick:hover {
		color: var(--neutral-700);
	}
	#ticks .plane {
		border-bottom: 4px solid var(--neutral-300);
		border-top: 0;
		margin-top: -18px;
		align-items: flex-end;
	}
</style>
