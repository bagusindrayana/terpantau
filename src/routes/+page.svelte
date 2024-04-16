<script>
	import Item from "../Item.svelte";
	import { onMount } from "svelte";
	import { page } from "$app/stores";
	import { PUBLIC_API_URL } from "$env/static/public"
	let curPage = 1;

	let categories;
	let data;
	const wilayahs = [
		"Tol",
		"Bali",
		"Bandung",
		"BanjarBaru",
		"Bekasi",
		"Cirebon",
		"Jogja",
		"KabBanjar",
		"KabOku",
		"Malang",
		"Samarinda",
	];

	/**
	 * @param {string} wilayah
	 */
	async function getData(wilayah) {
		return fetch(`${PUBLIC_API_URL}/cctv/${wilayah}?page=${curPage}`)
			.then((r) => r.json())
			.then((responseData) => {
				if (!responseData.customCategory) {
					categories = null;
				}
				data = responseData;
			});
	}

	function prev(){
		if(parseInt(curPage) > 1){
			curPage = parseInt(curPage)-1;
		}
		const wilayah = document.getElementById('wilayah').value;
		getData(wilayah);
	}

	function next(){
		curPage = parseInt(curPage)+1;
		const wilayah = document.getElementById('wilayah').value;
		getData(wilayah);
	}

	onMount(async () => {
		curPage = parseInt($page.url.searchParams.get("page")) || 1;
		await getData('Tol');

		const videos = document.querySelectorAll("video");
		videos.forEach((video) => {
			const source = video.getElementsByTagName("source")[0].src;

			// For more options see: https://github.com/sampotts/plyr/#options
			// captions.update is required for captions to work with hls.js
			const defaultOptions = {
				controls: [],
			};

			if (Hls.isSupported()) {
				// For more Hls.js options, see https://github.com/dailymotion/hls.js
				const hls = new Hls();
				hls.loadSource(source);

				// From the m3u8 playlist, hls parses the manifest and returns
				// all available video qualities. This is important, in this approach,
				// we will have one source on the Plyr player.
				hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
					// Initialize here
					const player = new Plyr(video, defaultOptions);
				});
				hls.attachMedia(video);
				window.hls = hls;
			} else {
				// default options with no quality update in case Hls is not supported
				const player = new Plyr(video, defaultOptions);
			}
		});
	});
</script>

<svelte:head>
	<title>TERPANTAU - Pantau CCTV!</title>
	<meta name="description" content="Pantau CCTV Seluruh Indonesia!" />
	<link rel="stylesheet" href="https://unpkg.com/plyr@3/dist/plyr.css" />
</svelte:head>

<section class="text-gray-600 body-font">
	<div class="container px-5 py-24 mx-auto">
		<div class="flex flex-col text-center w-full mb-20">
			<h1
				class="sm:text-3xl text-2xl font-medium title-font mb-4 text-gray-900"
			>
				Terpantau! - Pantau CCTV
			</h1>
			<p class="lg:w-2/3 mx-auto leading-relaxed text-base">
				stream CCTV bersumber dari pemerintah kota maupun provinsi yang
				tersedia secara publik.
			</p>
		</div>
		<div class="w-full flex justify-center mb-6">
			<select
				on:change={(e) => {
					data = null;
					getData(e.target.value);
				}}
				id="wilayah"
				class="bg-gray-50 border border-gray-300 "
			>
				{#each wilayahs as wilayah}
					<option value={wilayah}>{wilayah}</option>
				{/each}
			</select>
		</div>
		<div class="w-full flex justify-center mb-20">
			{#if data && data.paginate}
			<nav class="isolate inline-flex -space-x-px rounded-md shadow-sm" aria-label="Pagination">
				<a href="{curPage > 1 ? '?page='+curPage : '#'}" on:click={prev} class="relative inline-flex items-center rounded-l-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0">
				  <span class="sr-only">Previous</span>
				  <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
					<path fill-rule="evenodd" d="M12.79 5.23a.75.75 0 01-.02 1.06L8.832 10l3.938 3.71a.75.75 0 11-1.04 1.08l-4.5-4.25a.75.75 0 010-1.08l4.5-4.25a.75.75 0 011.06.02z" clip-rule="evenodd" />
				  </svg>
				</a>
				
				<a href="?page={curPage}" on:click={next} class="relative inline-flex items-center rounded-r-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0">
				  <span class="sr-only">Next</span>
				  <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
					<path fill-rule="evenodd" d="M7.21 14.77a.75.75 0 01.02-1.06L11.168 10 7.23 6.29a.75.75 0 111.04-1.08l4.5 4.25a.75.75 0 010 1.08l-4.5 4.25a.75.75 0 01-1.06-.02z" clip-rule="evenodd" />
				  </svg>
				</a>
			  </nav>

			{/if}
		</div>
		<div></div>
		<div class="flex flex-wrap -m-4">
			{#if data}
				{#each data.data as item}
					<Item {item} />
				{/each}
			{:else}
				<p class="loading">loading...</p>
			{/if}
		</div>
	</div>
</section>
