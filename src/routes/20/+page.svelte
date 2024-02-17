<script lang="ts">
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	onMount(() => {
		goto('https://flo-bit.github.io/turing-tinder/');
	});
</script>

<svelte:head>
	<meta name="twitter:image" content="https://flo-bit.github.io/turing-tinder/results/20.png" />
</svelte:head>
