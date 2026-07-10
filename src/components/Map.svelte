<script>
	import { range, visibleSpokes, selectedAirport } from '../lib/stores.js';
	import * as d3 from 'd3';
	import { mesh, feature } from 'topojson-client';

	let { map, airports } = $props();

	const projection = d3.geoAlbersUsa().scale(1300).translate([487.5, 305]);
	const path = d3.geoPath();
	const pathProjected = d3.geoPath(projection);
	const states = mesh(map, map.objects.states, (a, b) => a !== b);
	const nation = feature(map, map.objects.nation);

	let airportLocked = false;

	function lockAirport(iataCode) {
		if(iataCode === $selectedAirport) iataCode = undefined;
		airportLocked = true;
		$selectedAirport = iataCode;
	}

	function unlockAirport() {
		airportLocked = false;
		$selectedAirport = undefined;
	}

	function spokeColor({properties}) {
		if(properties.DISTANCE <= $range) return '#61A0FF';
		return '#FF8A00';
	}

	const spokeWidthScale = d3.scaleLinear().domain([1, 102, 315, 720, 2029]).range([.5,.75,1.25,3]);
	const spokeOpacityScale = d3.scaleLinear().domain([1, 15712, 47498, 102776, 248722]).range([.2, .3, .4, .5]);
	const spokeWidthModifier = d3.scaleLinear().domain([0,1000]).range([0,0]).clamp(true);
	const spokeOpacityModifier = d3.scaleLinear().domain([0,1000]).range([.25, -.1]).clamp(true);

	function spokeOpacity(passengers) {
		const opacity = spokeOpacityScale(passengers) + spokeOpacityModifier($range);
		if($visibleSpokes.spokes.length <= 20) return Math.max(0.5, opacity);
		return opacity;
	}

	let hoveredAirport = $state(null);
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<!-- svelte-ignore a11y_interactive_supports_focus -->
<div onclick={unlockAirport} role="button" aria-label="Deselect airport">
	<svg viewBox='18 13 940 595' preserveAspectRatio="xMinYMin meet" id="airports">
		<g fill="none" stroke-linejoin="round" stroke-linecap="round">
			<path stroke-width="1" stroke="var(--neutral-300)" fill="var(--neutral-50)" d="{path(nation)}" />
			<path stroke-width=".6" stroke="var(--neutral-300)" d="{path(states)}" />
		</g>

		<g fill="none" stroke="#000" stroke-linejoin="round" stroke-linecap="round" id="spokes">
			{#each $visibleSpokes.spokes as spoke (spoke.id)}
			<path
				class="spoke"
				stroke="{spokeColor(spoke)}"
				stroke-width="{ spokeWidthScale(spoke.properties.DEPARTURES_PERFORMED) + spokeWidthModifier($range) }"
				stroke-opacity="{ spokeOpacity(spoke.properties.PASSENGERS) }"
				d="{pathProjected(spoke)}"
			/>
			{/each}
		</g>

		<g fill="rgba(0,0,0,.25)" stroke-width="9" stroke="transparent" stroke-linejoin="round" stroke-linecap="round">
			{#each airports.features as airport}
				{@const isSelected = airport.properties.iata_code === $selectedAirport}
				{@const isHovered = airport.properties.iata_code === hoveredAirport}
				<circle
					cx="{projection(airport.geometry.coordinates)[0]}"
					cy="{projection(airport.geometry.coordinates)[1]}"
					r={isSelected ? "4" : "2.25"}
					onclick={(e) => { e.stopPropagation(); lockAirport(airport.properties.iata_code); }}
					onmouseenter={() => hoveredAirport = airport.properties.iata_code}
					onmouseleave={() => hoveredAirport = null}
					class:selected={isSelected}
					class:hovered={isHovered}
					role="button"
					tabindex="0"
				/>
				{#if isHovered || isSelected}
					<text
						x="{projection(airport.geometry.coordinates)[0]}"
						y="{projection(airport.geometry.coordinates)[1] - 8}"
						text-anchor="middle"
						class="airport-label"
					>{airport.properties.iata_code}</text>
				{/if}
			{/each}
			</g>
	</svg>
</div>

<style>
    div {
        position: relative;
    }

    svg {
        width: 100%;
        height: 100%;
    }

    #spokes {
        pointer-events: none;
    }

    #airports circle {
        cursor: pointer;
    }

    #airports circle:hover,
    #airports circle.hovered {
        fill: rgba(0,0,0,.7);
    }

    #airports circle.selected {
        fill: rgba(0,0,0,.7);
        transition: fill .25s;
    }

    .airport-label {
        font-size: 10px;
        font-weight: 600;
        fill: var(--neutral-700);
        pointer-events: none;
    }
</style>
