<script>
	import { onMount } from 'svelte';
	import { pokemonStore } from '../store';

	import { fade, fly, blur, slide, draw, crossfade } from 'svelte/transition';

	import Star from '$lib/icons/Star.svelte';
	import ChevronRight from '$lib/icons/ChevronRight.svelte';
	import Modal from '$lib/components/Modal.svelte';

	let numberOfPokemonsToFetch = 100;
	let numberOfPokemonsToShow = 9;
	let pokemons = [];
	let pokemonWithAdditionalInfo = 0;

	let isModalOpen = false;
	let isLoading = true;
	let pokemonToOpen;
	let searchInput = '';

	pokemonStore.subscribe((value) => {
		pokemonWithAdditionalInfo = value;
	});

	onMount(async () => {
		const res = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${numberOfPokemonsToFetch}`);
		const result = await res.json();
		pokemons = result.results;
		isLoading = false;

		if (pokemons.length > 0) {
			for (const pokemon of pokemons) {
				const singlePokemonFetch = await fetch(pokemon.url);
				const res = await singlePokemonFetch.json();
				pokemonStore.update((value) => [...value, res]);
			}
		}
	});

	const favouriteClick = (id) => {
		pokemonWithAdditionalInfo.forEach((pokemon) => {
			if (pokemon.id === id) {
				return (pokemon.isFavourite = !pokemon.isFavourite);
			}
		});
		pokemonStore.update(() => [...pokemonWithAdditionalInfo]);
	};

	const openPokemonModal = (id) => {
		pokemonToOpen = pokemonWithAdditionalInfo.filter((pokemon) => pokemon.id === id);
		isModalOpen = true;
	};

	const closePokemonModal = () => {
		isModalOpen = false;
		pokemonToOpen = [];
	};
	$: {
		if (searchInput) {
			pokemonWithAdditionalInfo = $pokemonStore.filter((pokemon) =>
				pokemon.name.toLowerCase().includes(searchInput.toLowerCase())
			);
			numberOfPokemonsToShow = 1;
		} else {
			pokemonWithAdditionalInfo = [...$pokemonStore];
		}
	}
</script>

<main class={`py-20 bg-gray-50 ${isLoading ? 'h-screen' : ''}`}>
	<h1 class="text-4xl font-bold text-center">
		<span class="text-gray-800"> Svelte</span>

		<span class="text-purple-600"> Pokedex</span>
	</h1>
	<div class=" mt-10 flex justify-center items-center ">
		<input
			type="text"
			bind:value={searchInput}
			class=" ring-1 border-purple-600 rounded-md py-2 w-80  focus:outline-none focus:ring-2 ring-purple-400 focus:ring-offset-2 pl-4 "
			placeholder="Search your pokemon "
		/>
	</div>

	<div class=" w-4/5 m-auto mt-10 grid sm:grid-cols-3 grid-cols-1 gap-20">
		{#if !isLoading}
			{#each pokemonWithAdditionalInfo.slice(0, numberOfPokemonsToShow) as pokemon}
				<div
					class="relative h-72 bg-white rounded-lg shadow-lg flex flex-col justify-between
					items-center overflow-hidden group"
					transition:slide={{ x: 200, duration: 2500 }}
				>
					<img
						src={pokemon['sprites']['other']['official-artwork']['front_default']}
						alt={pokemon.name}
						class="w-40 mt-4"
					/>
					<div
						class="absolute h-full w-full transform translate-y-3/4 p-4 bg-purple-600 text-center text-white transition-all duration-700 ease-in-out group-hover:translate-y-0"
					>
						{#if pokemon.isFavourite}
							<Star
								favouriteClick={() => favouriteClick(pokemon.id)}
								additionalClass="absolute top-5 left-5 fill-current  text-yellow-500 transition-all duration-700 ease-in-out hover:animate-pulse hover:cursor-pointer group-hover:text-opacity-100 "
							/>
						{:else}
							<Star
								favouriteClick={() => favouriteClick(pokemon.id)}
								additionalClass="absolute top-5 left-5  stroke-current text-white text-opacity-0 transition-all duration-700 ease-in-out hover:animate-pulse hover:cursor-pointer group-hover:text-opacity-100 "
							/>
						{/if}
						<p class="text-2xl font-bold capitalize">
							{pokemon.name}
						</p>
						<ChevronRight
							favouriteClick={() => openPokemonModal(pokemon.id)}
							additionalClass="absolute top-5 right-5 stroke-current text-white text-opacity-0 transition-all duration-700 ease-in-out hover:animate-pulse hover:cursor-pointer group-hover:text-opacity-100 
              
        "
						/>
						<div class="mt-10 w-8/12 m-auto">
							{#each pokemon.stats as stat}
								<p class="flex justify-between text-base border-b-2 mb-1">
									<span class="font-bold capitalize">{stat.stat.name}</span>
									<span>{stat.base_stat}</span>
								</p>
							{/each}
						</div>
					</div>
				</div>
			{/each}
		{:else}
			<div class="relative h-72 bg-gray-300 rounded-lg shadow-lg animate-pulse" />
			<div class="relative h-72 bg-gray-300 rounded-lg shadow-lg animate-pulse" />
			<div class="relative h-72 bg-gray-300 rounded-lg shadow-lg animate-pulse" />
		{/if}
	</div>
	{#if !isLoading && numberOfPokemonsToShow < pokemonWithAdditionalInfo.length}
		<div class=" mt-10 flex justify-center items-center ">
			<button
				on:click={() => (numberOfPokemonsToShow = numberOfPokemonsToShow + 9)}
				type="button"
				class="py-2 px-4 rounded bg-purple-600 text-white font-bold transition-all duration-700 ease-in-out hover:bg-purple-900 focus:outline-none"
			>
				Load More
			</button>
		</div>
	{/if}
	{#if isModalOpen}
		<Modal {closePokemonModal} pokemonToOpen={pokemonToOpen[0]} />
	{/if}
</main>
