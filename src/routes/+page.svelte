<script>
	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';
	import Nav from '$lib/components/Nav.svelte';
</script>

<main>
	<Header />
	<Nav />
	<Footer />
</main>

<style>
	main {
		background-color: aliceblue;
		max-width: 980px;
		margin: 0 auto;
		padding-bottom: 10px;
	}
</style>
