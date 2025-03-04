<script>
	import { onMount } from 'svelte';
	import { feature } from 'topojson-client';
	import bbox from '@turf/bbox';

	import { Map, MapSource, MapLayer, MapTooltip } from '@onsvisual/svelte-maps';

	import { geoTypes, countries, path, style } from '$lib/config.js';
	import years from '$lib/years.js';

	let hovered;
	let bounds;
	let lookup;

	let geojson = {};
	let geoType = geoTypes[0];
	let year = years[years.length - 1];
	let ctrys = [...countries];

	let map = null;

	function filterGeo(geo, year, ctrys) {
		let filtered = JSON.parse(JSON.stringify(geo));
		let codes = ['K', ...ctrys.map((c) => c.key)];
		filtered.features = filtered.features
			.filter((f) => {
				return (
					codes.includes(f.properties.areacd[0]) &&
					!(f.properties.end && f.properties.end < year) &&
					!(f.properties.start && f.properties.start > year)
				);
			})
			.map((f) => {
				f.properties = f.properties = { areacd: f.properties.areacd, areanm: f.properties.areanm };
				return f;
			});
		return filtered;
	}
	$: filteredGeo = bounds ? filterGeo(geojson[geoType.key], year, ctrys) : null;

	async function init() {
		const topojson = await (await fetch(path)).json();

		let lkp = {};
		geoTypes.forEach((type) => {
			geojson[type.key] = feature(topojson, topojson.objects[type.key]);
			geojson[type.key].features.forEach((f) => (lkp[f.properties.areacd] = f.properties));
		});
		bounds = bbox(geojson['uk']);
		lookup = lkp;
	}
	onMount(init);
</script>

{#if filteredGeo}
	<div class="map-container">
		<Map bind:map {style} controls location={{ bounds }}>
			<MapSource id="geo" type="geojson" data={filteredGeo} promoteId="areacd">
				<MapLayer
					id="geo-fill"
					type="fill"
					paint={{
						'fill-color': 'white',
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
						'line-color': 'white',
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
