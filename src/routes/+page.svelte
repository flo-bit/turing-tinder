<script lang="ts">
	import { CardSwiper } from '$lib/CardSwiper';
	import { fade } from 'svelte/transition';

	import TwitterShare from '$lib/TwitterShare.svelte';
	import confetti from 'canvas-confetti';

	const launchConfetti = () => {
		let count = 200;
		let defaults = {
			origin: { y: 0.7 }
		};

		function fire(particleRatio: number, opts: confetti.Options | undefined) {
			confetti({
				...defaults,
				...opts,
				particleCount: Math.floor(count * particleRatio)
			});
		}

		fire(0.25, {
			spread: 26,
			startVelocity: 55
		});
		fire(0.2, {
			spread: 60
		});
		fire(0.35, {
			spread: 100,
			decay: 0.91,
			scalar: 0.8
		});
		fire(0.1, {
			spread: 120,
			startVelocity: 25,
			decay: 0.92,
			scalar: 1.2
		});
		fire(0.1, {
			spread: 120,
			startVelocity: 45
		});
	};

	let swipe: (direction?: 'left' | 'right') => void;

	let thresholdPassed = 0;

	let correctGuess: boolean | undefined = undefined;

	let allGuesses = 0;
	let correctGuesses = 0;
	let wrongGuesses = 0;

	let showMenu = true;

	let aiImages = Array.from({ length: 10 }, (_, i) => `/turing-tinder/ai/${i}.webp`);
	let humanImages = Array.from({ length: 10 }, (_, i) => `/turing-tinder/human/${i}.webp`);

	let allImages = aiImages.concat(humanImages);
	// shuffle the images
	for (let i = allImages.length - 1; i > 0; i--) {
		const j = Math.floor(Math.random() * (i + 1));
		[allImages[i], allImages[j]] = [allImages[j], allImages[i]];
	}

	let index = 1;

	$: percentage = (correctGuesses / allGuesses) * 100;

	let showScoreScreen = false;

	$: endText =
		percentage < 30
			? 'That was terrible!'
			: percentage < 60
				? 'Basically random guessing'
				: 'Well done!';
</script>

<svelte:head>
	<meta name="twitter:image" content="https://flo-bit.github.io/turing-tinder/results/0.png" />
</svelte:head>


<svelte:body
	on:keydown={(e) => {
		if (showMenu || showScoreScreen) return;

		if (e.key === 'ArrowLeft') {
			swipe('left');
		} else if (e.key === 'ArrowRight') {
			swipe('right');
		}
	}}
/>

<div class="h-[100svh] w-[100svw] p-2 flex items-center justify-center overflow-hidden bg-gray-950">
	<div class="w-full h-full max-w-xl relative">
		<CardSwiper
			bind:swipe
			cardData={(index) => {
				let image = allImages[index % allImages.length];
				return {
					image,
					bot: aiImages.includes(image)
				};
			}}
			on:swiped={(e) => {
				if (e.detail.data.bot === (e.detail.direction === 'left')) {
					correctGuess = true;
					correctGuesses++;
				} else {
					correctGuess = false;
					wrongGuesses++;
				}
				allGuesses++;
				if (allGuesses === 20) {
					showScoreScreen = true;

					launchConfetti();
				}

				setTimeout(() => {
					correctGuess = undefined;
				}, 400);

				index = e.detail.index + 2;
			}}
			bind:thresholdPassed
		/>

		<button
			class="absolute bottom-1 left-1 p-3 px-4 bg-rose-300/50 backdrop-blur-sm rounded-full z-10 text-3xl hover:opacity-70 ring-1 ring-rose-500"
			on:click={() => swipe('left')}
		>
			🤖
		</button>
		<button
			class="absolute bottom-1 right-1 p-3 px-4 bg-indigo-300/50 backdrop-blur-sm rounded-full z-10 text-3xl hover:opacity-70 ring-1 ring-indigo-500"
			on:click={() => swipe('right')}
		>
			🧑‍🎨
		</button>

		<!-- {#if !isNaN(percentage)}
			<div
				class="absolute top-0 left-0 right-0 w-full text-center text-xl text-black flex justify-center"
			>
				<div class="flex space-x-2 bg-white/50 backdrop-blur-sm rounded-md p-2">
					<div class="text-green-800">
						{percentage.toFixed(0)}%
					</div>
				</div>
			</div>
		{/if} -->
	</div>
	<!-- show percentage done as a line -->
	<div
		class="absolute bottom-0 left-0 right-0 w-full h-1 bg-red-500 transition-transform duration-200"
		style="width: {(allGuesses / 20) * 100}%"
	/>

	{#if thresholdPassed !== 0}
		<div
			transition:fade={{ duration: 200 }}
			class="absolute w-full h-full inset-0 {thresholdPassed < 0
				? 'bg-rose-200/20'
				: 'bg-indigo-200/20'} backdrop-blur-sm flex items-center justify-center text-9xl pointer-events-none"
		>
			<div class="flex flex-col items-center relative">
				{thresholdPassed > 0 ? '🧑‍🎨' : '🤖'}
				{#key correctGuess}
					{#if correctGuess !== undefined}
						<div class="absolute flex items-center justify-center w-full h-full">
							{#if correctGuess}
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="3.5"
									stroke="currentColor"
									class="w-full h-full text-green-500"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="m4.5 12.75 6 6 9-13.5" />
								</svg>
							{:else}
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="3.5"
									stroke="currentColor"
									class="w-full h-full text-red-500"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
								</svg>
							{/if}
						</div>
					{/if}
				{/key}
			</div>
		</div>
	{/if}

	{#if showMenu}
		<div
			transition:fade={{ duration: 200 }}
			class="absolute w-full h-full inset-0 bg-gray-950/70 text-gray-50 backdrop-blur-sm flex flex-col items-center justify-center text-9xl z-50"
		>
			<div class="flex flex-col max-w-sm text-center mx-4">
				<div class="text-5xl font-semibold mb-8">Turing Tinder</div>
				<div class="text-xl font-semibold mb-2">
					Can you tell which pictures where <span class="font-bold text-rose-300"
						>made by an AI</span
					>
					and which <span class="font-bold text-indigo-300">by a human</span>?
				</div>
				<div class="text-sm mb-8">Swipe left for AI, right for human. 20 pictures.</div>
			</div>
			<button
				type="button"
				on:click={() => (showMenu = false)}
				class="rounded-md bg-rose-500 px-5 py-2.5 text-2xl font-semibold text-white shadow-sm hover:bg-rose-400 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-500"
				>Let's go!</button
			>
		</div>
	{/if}

	{#if showScoreScreen}
		<div
			transition:fade={{ duration: 200 }}
			class="absolute w-full h-full inset-0 bg-gray-950/70 text-gray-50 backdrop-blur-sm flex flex-col items-center justify-center z-50"
		>
			<div class="flex flex-col max-w-sm text-center mx-4">
				<div class="text-5xl font-semibold mb-8">{endText}</div>
				<div class="text-8xl font-semibold mb-8">
					{percentage.toFixed(0)}%
				</div>
			</div>

			<TwitterShare
				text="I just played Turing Tinder and got a score of {percentage.toFixed(0)}%! Can you beat me? 🤖🧑‍🎨"
				url="https://flo-bit.github.io/turing-tinder/{percentage.toFixed(0)}?547849"
				hashtags="aiORhuman,turingtinder"
				via="flobit_dev"
				related="sveltejs"
			/>

			<!-- <button
				on:click={() => {
					showScoreScreen = false;
					allGuesses = 0;
					correctGuesses = 0;
					wrongGuesses = 0;
					index = 1;
				}}
				type="button"
				class="mt-4 rounded-md bg-white/10 px-3.5 py-2.5 text-sm font-semibold text-rose-500 shadow-sm hover:bg-white/20"
				>Try again</button
			> -->
		</div>
	{/if}
</div>
