<script lang="ts">
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	onMount(() => {
		goto('/turing-tinder');
	});
</script>

<svelte:head>
	<meta name="twitter:image" content="https://github.io/turing-tinder/results/20.png" />
</svelte:head>
