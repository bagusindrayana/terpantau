<script>
	import Item from "../Item.svelte";
	import { onMount } from "svelte";
	import { page } from "$app/stores";
	import { goto } from "$app/navigation";
	import { PUBLIC_API_URL } from "$env/static/public";

	/**
	 * @type {number}
	 */
	let curPage = 1;

	/**
	 * @type {null|Array<Object>}
	 */
	let categories = null;

	/**
	 * @type {null|String}
	 */
	let selectedCategory = null;

	/**
	 * @type {Object|null}
	 */
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
	 * @type {string}
	 */
	let selectedWilayah = "Tol";

	/**
	 * @param {string} wilayah
	 * @param {string|null} cat
	 */
	async function getData(wilayah, cat = null) {
		let url = `${PUBLIC_API_URL}/cctv/${wilayah}?page=${curPage}`;
		if(cat){
			url += `&category=${cat}`;
		}
		await fetch(
			`${url}`,
		)
			.then((r) => r.json())
			.then((responseData) => {
				data = responseData;
			});
		return Promise.resolve();
	}

	function prev() {
		if (curPage > 1) {
			curPage = curPage - 1;
		}
		data = null;
		updateQuqery();
		getData(selectedWilayah);
	}

	function next() {
		curPage = curPage + 1;
		data = null;
		updateQuqery();
		getData(selectedWilayah);
	}

	const updateQuqery = () => {
		$page.url.searchParams.set("wilayah", selectedWilayah);
		$page.url.searchParams.set("page", curPage.toString());
		goto(`?${$page.url.searchParams.toString()}`);
	};

	/**
	 * @param {Event} e
	 */
	function selectWilayah(e) {
		data = null;
		if (e.target) {
			selectedWilayah = e.target.value;
			curPage = 1;
			getData(e.target.value);
			//update url
			updateQuqery();
		}
	}

	/**
	 * @param {Event} e
	 */
	function selectCategory(e) {
		data = null;
		if (e.target) {
			selectedCategory = e.target.value;
			curPage = 1;
			getData(selectedWilayah, selectedCategory);
			//update url
			updateQuqery();
		}
	}

	const getCategory = () => {
		return fetch(`${PUBLIC_API_URL}/cctv/${selectedWilayah}/category`)
			.then((r) => r.json())
			.then((responseData) => {
				categories = responseData.data;
			});
	};

	onMount(async () => {
		curPage = parseInt($page.url.searchParams.get("page") || "1");
		const w = $page.url.searchParams.get("wilayah");
		if (w) {
			selectedWilayah = w;
		} else {
		}
		await getData(selectedWilayah);

		if (!data.customCategory) {
			categories = null;
		} else {
			getCategory();
		}
	});
</script>

<svelte:head>
	<title>TERPANTAU - Pantau CCTV!</title>
	<meta name="description" content="Pantau CCTV Seluruh Indonesia!" />
	<link rel="stylesheet" href="https://unpkg.com/plyr@3/dist/plyr.css" />

	<style>
		.plyr__controls {
			display: none;
		}
	</style>
</svelte:head>

<section class="text-gray-600 body-font">
	<div class="container px-5 py-12 md:py-24 mx-auto">
		<div class="flex flex-col text-center w-full mb-10 md:mb-20">
			<h1
				class="sm:text-3xl text-2xl font-medium title-font mb-4 text-gray-900"
			>
				Terpantau! - Pantau CCTV
			</h1>
			<p class="lg:w-2/3 mx-auto leading-relaxed text-base">
				stream CCTV bersumber dari website pemerintah kota maupun
				provinsi indonesia yang tersedia secara publik.
			</p>
		</div>
		<div class="w-full flex justify-center mb-3 md:mb-6">
			<select
				on:change={selectWilayah}
				id="wilayah"
				value={selectedWilayah}
				class="bg-gray-50 border border-gray-300"
			>
				{#each wilayahs as wilayah}
					<option value={wilayah}>{wilayah}</option>
				{/each}
			</select>
		</div>
		<div
			class="w-full flex flex-row justify-center items-center mb-10 md:mb-20"
		>
			{#if data && data.paginate}
				<nav
					class="isolate inline-flex -space-x-px rounded-md shadow-sm"
					aria-label="Pagination"
				>
					<button
						on:click={prev}
						class="relative inline-flex items-center rounded-l-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0"
					>
						<span class="sr-only">Previous</span>
						<svg
							class="h-5 w-5"
							viewBox="0 0 20 20"
							fill="currentColor"
							aria-hidden="true"
						>
							<path
								fill-rule="evenodd"
								d="M12.79 5.23a.75.75 0 01-.02 1.06L8.832 10l3.938 3.71a.75.75 0 11-1.04 1.08l-4.5-4.25a.75.75 0 010-1.08l4.5-4.25a.75.75 0 011.06.02z"
								clip-rule="evenodd"
							/>
						</svg>
					</button>

					<button
						on:click={next}
						class="relative inline-flex items-center rounded-r-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0"
					>
						<span class="sr-only">Next</span>
						<svg
							class="h-5 w-5"
							viewBox="0 0 20 20"
							fill="currentColor"
							aria-hidden="true"
						>
							<path
								fill-rule="evenodd"
								d="M7.21 14.77a.75.75 0 01.02-1.06L11.168 10 7.23 6.29a.75.75 0 111.04-1.08l4.5 4.25a.75.75 0 010 1.08l-4.5 4.25a.75.75 0 01-1.06-.02z"
								clip-rule="evenodd"
							/>
						</svg>
					</button>
				</nav>
			{/if}

			{#if categories}
				<select
					on:change={selectCategory}
					id="category"
					value={selectedCategory}
					class="bg-gray-50 border border-gray-300"
				>
					{#each categories as category}
						<option value={category.id}>{category.name}</option>
					{/each}
				</select>
			{/if}
		</div>
		<div></div>
		<div class="flex flex-wrap -m-4">
			{#if data}
				{#each data.data as item}
					<Item {item} source={data.source} />
				{/each}
			{:else}
				<p class="loading">loading...</p>
			{/if}
		</div>
	</div>
</section>
