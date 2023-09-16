<script>
	import { Table } from '@skeletonlabs/skeleton';
	import { ProgressRadial } from '@skeletonlabs/skeleton';
	import { onMount } from 'svelte';
	import { PUBLIC_BASE_URL } from '$env/static/public';

	/**
	 * @type {any[]}
	 */
	let sourceData = [];
	let page = 1;
	let count = 10;
	let isLoadingFind = false;
	let isLoadingMore = false;

	onMount(() => {
		isLoadingFind = false;
		isLoadingMore = false;
	});

	/**
	 * @param {any[]} data
	 * @param {string[]} keys
	 */
	function tableMapperValues(data, keys) {
		return data.map((item) => {
			return keys.map((key) => {
				if ((key === 'url' || key === 'video') && item[key]) {
					return `<a href="${item[key]}" target="_blank">${item[key]}</a>`;
				} else {
					return item[key];
				}
			});
		});
	}

	async function getPromoResponse() {
		const inputElement = document.getElementById('searchInput');
		const productName = inputElement?.value ?? '';

		// Define the URL with query parameters
		const url = new URL(PUBLIC_BASE_URL + 'api/promotions');
		url.searchParams.append('productName', productName);
		url.searchParams.append('page', page.toString());
		url.searchParams.append('count', count.toString());

		// Create a fetch request with headers
		const response = await fetch(url.toString(), {
			method: 'GET'
		});

		if (!response.ok) {
			throw new Error('Network response was not ok');
		}
		return response;
	}

	async function fetchDataInitalSourceData() {
		isLoadingFind = true;
		page = 1;
		const response = await getPromoResponse();
		try {
			sourceData = await response.json();
			updateTable();
			page += 1;
		} catch (error) {
			console.error('Error:', error);
		}
		isLoadingFind = false;
	}

	async function getMorePromotions() {
		if (sourceData.length == 0 || sourceData.length > 100) {
			return;
		}
		isLoadingMore = true;
		try {
			const response = await getPromoResponse();
			var newData = await response.json();
			if (newData.length > 0) {
				page += 1;
			}
			sourceData.push(...newData);
			updateTable();
		} catch (error) {
			console.error('Error:', error);
		}
		isLoadingMore = false;
	}

	// Function to update the table when data changes
	function updateTable() {
		tableSimple.body = tableMapperValues(sourceData, [
			'product',
			'code',
			'url',
			'video',
			'channel'
		]);
	}

	const tableSimple = {
		head: ['Product', 'Code', 'Link', 'Video', 'Channel'],
		body: tableMapperValues(sourceData, ['product', 'code', 'url', 'video', 'channel']),
		meta: tableMapperValues(sourceData, ['product', 'code', 'url', 'video', 'channel']),
		foot: []
	};

	/**
	 * @param {{ key: string; }} event
	 */
	function handleKeyPress(event) {
		if (event.key === 'Enter') {
			fetchDataInitalSourceData();
		}
	}
</script>

<html lang="en">
	<h1 class="h1">
		<span
			class="bg-gradient-to-br from-blue-500 to-cyan-300 bg-clip-text text-transparent box-decoration-clone"
			>Promotion and Codes</span
		>
	</h1>
	<div class="center-container">
		<div class="center">
			<div class="input-container">
				<input
					id="searchInput"
					class="input"
					title="Input (search)"
					type="search"
					placeholder="product name"
					on:keydown={handleKeyPress}
				/>
				<button
					on:click={() => {
						if (document.getElementById('searchInput').value.trim() !== '') {
							fetchDataInitalSourceData();
						}
					}}
					type="button"
					class="btn variant-ghost-surface"
				>
					{#if isLoadingFind}
						<ProgressRadial
							stroke={200}
							meter="stroke-primary-500"
							track="stroke-primary-500/30"
							class="w-5"
						/>
					{:else}
						Find
					{/if}
				</button>
			</div>

			<div class="table">
				<Table source={tableSimple} />
			</div>
			<div class="center-button">
				<button
					on:click={() => getMorePromotions()}
					type="button"
					class="btn variant-ghost-surface"
				>
					{#if isLoadingMore}
						<ProgressRadial
							stroke={200}
							meter="stroke-primary-500"
							track="stroke-primary-500/30"
							class="w-5"
						/>
					{:else}
						More
					{/if}
				</button>
			</div>
		</div>
	</div>
</html>

<style lang="css">
	/* thanks chat-gpt :) */
	.h1 {
		margin-top: 5%;
		margin-left: 40%;
		margin-bottom: 3%;
	}
	.center-container {
		justify-content: center;
		margin: 0 20%; /* 20% margin on both sides */
	}
	.center-button {
		justify-content: center;
		margin: 0 50%;
	}
	.input-container {
		display: flex;
		align-items: center; /* Vertically center items */
		margin-right: 60%;
		margin-top: 20px;
		margin-bottom: 20px;
	}
	.table {
		margin-bottom: 20px;
	}

	.input {
		flex-grow: 1; /* Grow to fill available space */
		margin-right: 10px; /* Add some spacing between input and button */
	}
</style>
