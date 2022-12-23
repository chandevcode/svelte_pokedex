<script>
	export let additionalClass;
	export let favouriteClick;
</script>

<svg
	on:click={favouriteClick}
	xmlns="http://www.w3.org/2000/svg"
	class={`h-6 w-6 ${additionalClass}`}
	fill="none"
	viewBox="0 0 24 24"
>
	<path
		stroke-linecap="round"
		stroke-linejoin="round"
		stroke-width="2"
		d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z"
	/>
</svg>
