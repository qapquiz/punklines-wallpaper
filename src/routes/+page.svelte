<script lang="ts">
	import { onMount } from 'svelte';

	export let background = "#000";

	let canvas: HTMLCanvasElement;
	let context: CanvasRenderingContext2D | null;

	onMount(() => {
		context = canvas.getContext("2d");
	});
</script>

<div class="flex-1 flex flex-col justify-center items-center gap-4">
	<h1 class="font-mono text-4xl bg-gradient-to-r from-[#DC1FFF] via-[#03E1FF] to-[#00FFA3] inline-block text-transparent bg-clip-text">PUNKLINES</h1>
	<h2 class="font-mono text-xl bg-gradient-to-r from-[#DC1FFF] via-[#03E1FF] to-[#00FFA3] inline-block text-transparent bg-clip-text">Wallpaper Generator</h2>
	<canvas
		class="aspect-[9/16] h-3/4 rounded-md"
		style:background
		bind:this={canvas}
	/>
	<div class="flex flex-col gap-4">
		<input
			type="text"
			placeholder="PUNKLINES no."
			class="font-mono input input bordered input-accent"
		/>
		<button class="font-mono btn btn-accent">GENERATE</button>
	</div>
</div>
