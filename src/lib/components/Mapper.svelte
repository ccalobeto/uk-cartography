<script>
	import { Map, MapSource, MapLayer, MapTooltip } from '@onsvisual/svelte-maps';

	import { style } from '$lib/config.js';

	export let filteredGeo;
	export let bounds;
	export let lookup;

	let hovered;

	let map = null;
</script>

{#if filteredGeo}
	<div class="map-container">
		<Map bind:map {style} controls location={{ bounds }}>
			<MapSource id="geo" type="geojson" data={filteredGeo} promoteId="areacd">
				<MapLayer
					id="geo-fill"
					type="fill"
					paint={{
						'fill-color': 'gray',
						'fill-opacity': ['case', ['==', ['feature-state', 'hovered'], true], 0.3, 0]
					}}
					hover
					bind:hovered
				>
					<MapTooltip content={hovered ? `<b>${lookup[hovered].areanm}</b><br/>${hovered}` : ''} />
				</MapLayer>
				<MapLayer
					id="geo-line"
					type="line"
					paint={{
						'line-color': 'gray',
						'line-width': ['case', ['==', ['feature-state', 'hovered'], true], 2.5, 0.7]
					}}
				/>
			</MapSource>
		</Map>
	</div>
{/if}

<style>
	.map-container {
		height: 500px;
	}
</style>
