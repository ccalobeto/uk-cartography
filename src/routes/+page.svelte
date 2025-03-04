<script>
	import { topology } from 'topojson-server';
	import { csvFormat } from 'd3-dsv';

	import { coordEach } from '@turf/meta';
	import { stringify } from 'wkt';

	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';
	import Nav from '$lib/components/Nav.svelte';
	import Mapper from '$lib/components/Mapper.svelte';
</script>

<main>
	<Header />
	<Nav />
	<Mapper />

	<Footer />
</main>

<style>
	main {
		background-color: pink;
		border: 2px solid red;
		max-width: 980px;
		margin: 0 auto;
		padding-bottom: 10px;
	}
</style>
