<script>
	
	import "../app.css";
    import Navbar from "../components/Navbar.svelte";
</script>

<div class="app">
	
	<main>
		<Navbar />
		<slot />
	</main>

	
</div>
