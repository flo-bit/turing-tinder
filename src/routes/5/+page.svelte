<script lang="ts">
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	onMount(() => {
		goto('/turing-tinder');
	});
</script>

<svelte:head>
	<meta name="twitter:image" content="https://flo-bit.github.io/turing-tinder/twitter/ai-human-5.png" />
</svelte:head>
