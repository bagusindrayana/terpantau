<script>
    import Player from "../../../components/Player.svelte";
    import { page } from "$app/stores";

    var streamInfo = JSON.parse(atob($page.url.searchParams.get("stream")));
</script>
<svelte:head>
	<title>IFRAME!</title>
	
	<link rel="stylesheet" href="https://unpkg.com/plyr@3/dist/plyr.css" />

	<style>
		.plyr__controls {
			display: none;
		}
	</style>
</svelte:head>
<div class="relative w-full" style="height: 100vh;">
    <Player streamInfo={streamInfo}  preview={true}/>
</div>
