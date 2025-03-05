<script>
	import { onMount } from 'svelte';
	import { feature } from 'topojson-client';
	import { topology } from 'topojson-server';
	import { coordEach } from '@turf/meta';
	import bbox from '@turf/bbox';
	import { csvFormat } from 'd3-dsv';
	import { stringify } from 'wkt';

	import Mapper from '$lib/components/Mapper.svelte';

	import { geoTypes, countries, path } from '$lib/config.js';
	import years from '$lib/years.js';

	let geojson = {};
	let bounds;
	let lookup;

	const areaCode = 'id';
	const areaName = 'name';

	let geoType = geoTypes[0];
	let year = years[years.length - 1];
	let ctrys = [...countries];

	function download(mode = 'topo') {
		const key = geoType.key;
		const geo = {};
		geo[key] = filteredGeo;

		let output, filename;

		if (mode !== 'topo') {
			output = geo[key];
			coordEach(output, (c) => {
				c[0] = Math.round(c[0] * 1e4) / 1e4;
				c[1] = Math.round(c[1] * 1e4) / 1e4;
			});
			if (mode === 'csv') {
				output = csvFormat(
					output.features
						.map((f) => ({ ...f.properties, geometry: stringify(f.geometry) }))
						.sort((a, b) => a[areaCode].localeCompare(b[areaCode]))
				);
				filename = `${key}${year}.csv`;
			} else {
				filename = `${key}${year}.geojson`;
			}
		} else {
			output = topology(geo, 1e5);
			filename = `${key}${year}.json`;
		}

		const blob =
			mode === 'csv'
				? new Blob([output], { type: 'text/csv' })
				: new Blob([JSON.stringify(output)], { type: 'application/json' });
		const url = window.URL || window.webkitURL || window;
		const link = url.createObjectURL(blob);
		const a = document.createElement('a');

		a.download = filename;
		a.href = link;
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
	}

	function filterGeo(geo, year, ctrys) {
		let filtered = JSON.parse(JSON.stringify(geo));
		let codes = ['K', ...ctrys.map((c) => c.key)];
		filtered.features = filtered.features.map((f) => {
			f.properties = f.properties = {
				areacd: f.properties[areaCode],
				areanm: f.properties[areaName]
			};
			return f;
		});
		return filtered;
	}

	async function init() {
		const topojson = await (await fetch(path)).json();

		let lkp = {};
		geoTypes.forEach((type) => {
			geojson[type.key] = feature(topojson, topojson.objects[type.key]);
			geojson[type.key].features.forEach((f) => (lkp[f.properties[areaCode]] = f.properties));
		});

		bounds = bbox(geojson['level1']);
		lookup = lkp;
	}
	onMount(init);

	$: filteredGeo = bounds ? filterGeo(geojson[geoType.key], year, ctrys) : null;
</script>

<nav>
	<div>
		<label>
			Geography type<br />
			<select bind:value={geoType}>
				{#each geoTypes as type}
					<option value={type}>{type.label}</option>
				{/each}
			</select>
		</label>

		<label>
			Year<br />
			<select bind:value={year}>
				{#each years as y}
					<option value={y}>{y}</option>
				{/each}
			</select>
		</label>

		<div class="countries">
			Countries<br />
			{#each countries as c}
				<label>
					<input type="checkbox" name="countries" value={c} bind:group={ctrys} />
					{c.label}
				</label>
			{/each}
		</div>
	</div>
	<div>
		<button on:click={() => download('geo')}> Download GeoJSON </button>
		<button on:click={() => download('topo')}> Download TopoJSON </button>
		<button on:click={() => download('csv')}> Download CSV/WKT </button>
	</div>
</nav>

<Mapper {bounds} {lookup} {filteredGeo} />

<style>
	nav > div {
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		align-items: flex-start;
	}
	button,
	select {
		height: 40px;
		margin: 0 10px 10px 0;
	}
	input {
		margin-bottom: 16px;
	}

	.countries > label {
		display: inline-block;
		margin-right: 12px;
	}
</style>
