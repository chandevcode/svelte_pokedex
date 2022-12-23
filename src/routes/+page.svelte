<script>
	import { onMount } from 'svelte';
	import { pokemonStore } from '../store';

	import Start from '$lib/icons/Star.svelte';
	import ChevronRight from '$lib/icons/ChevronRight.svelte';
	import Modal from '$lib/components/Modal.svelte';

	let numberOfPokemonsToFetch = 100;
	let numberOfPokemonsToShow = 12;
	let pokemons = [];
	let pokemonWithAdditionalInfo;
	let isModalOpen = false;
	let isLoading = true;
	let pokemonToOpen;

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
				const singlePokemonInfo = await singlePokemonFetch.json();
				pokemonStore.update((value) => [...value, singlePokemonInfo]);
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
</script>
