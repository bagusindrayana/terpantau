<script>
	import "../app.css";
	import { inject } from '@vercel/analytics'
	import { onMount } from 'svelte';

	onMount(() => {
		inject();
	})
    
</script>

<div class="app">
	
	<slot/>

	
</div>
