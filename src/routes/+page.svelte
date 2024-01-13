<script lang="ts">
	import { Effect, pipe } from "effect";

	export let background = "#000";

	let canvas: HTMLCanvasElement;
	let punklinesNumberInput = "";
	let isLoading = false;
	let shownImageBitmap: ImageBitmap | null = null;

	const debounce = <T extends (...args: any[]) => void>(func: T, delay: number) => {
		let timeoutId: ReturnType<typeof setTimeout>;

		return (...args: Parameters<T>) => {
			clearTimeout(timeoutId);
			timeoutId = setTimeout(() => {
				func(...args);
			}, delay);
		};
	};

	const debouncedInputChange = debounce((event: Event) => {
		if (Number(punklinesNumberInput) > 4999) {
			punklinesNumberInput = '4999';
		}

		Effect.runPromise(generate(punklinesNumberInput)).then(console.log, console.error);
	}, 700);


	function downloadImage() {
		if (!shownImageBitmap) {
			return;
		}

		const newCanvas = document.createElement('canvas');
		newCanvas.width = 596;
		newCanvas.height = 1292;
		const newContext = newCanvas.getContext("2d");
		if (!newContext) {
			return;
		}

		newContext.fillStyle = '#000';
		newContext.fillRect(0, 0, newCanvas.width, newCanvas.height)
		newContext.drawImage(shownImageBitmap, 0, newCanvas.height - newCanvas.width, newCanvas.width, newCanvas.width);

		newCanvas.toBlob((blob) => {
			if (!blob) {
				return;
			}

			const imageDataUrl = URL.createObjectURL(blob)

			const link = document.createElement("a");
			link.href = imageDataUrl;
			link.download = `punkline_${punklinesNumberInput}_wallpaper.png`;
			link.click();
		});

	}

	function getPunklinesImage(
		punklinesNumber: string,
	): Effect.Effect<never, Error, Blob> {
		return Effect.tryPromise({
			try: async () => {
				console.log(punklinesNumber)
				if (punklinesNumber === '' || Number(punklinesNumber) > 4999) {
					throw new Error("punkline number should not exceed 4999");
				}

				isLoading = true;
				const imageUri = `https://gateway.pinit.io/ipfs/QmcZBboZZtnGtLMPoFq8VDdjcSTxy5FDMciRogW7An7W7B/${punklinesNumber}`;
				const response = await fetch(imageUri);
				const blobImage = await response.blob();
				isLoading = false;

				return blobImage;

				// return URL.createObjectURL(blobImage) as ObjectURL;
			},
			catch: () => {
				isLoading = false;
				return new Error("cannot get punkline image");
			}
		})
	}


	function generate(punklineNumber: string): Effect.Effect<never, Error, ImageBitmap> {
		return pipe(
			Effect.succeed(punklineNumber),
			Effect.flatMap(getPunklinesImage),
			Effect.flatMap((blob: Blob) => {
				return Effect.tryPromise({
					try: () => createImageBitmap(blob),
					catch: () => new Error("cannot create ImageBitmap")
				});
			}),
			Effect.tap((imageBitmap: ImageBitmap) => {
				const context = canvas?.getContext("2d");
				context?.drawImage(imageBitmap, 0, canvas.height - canvas.width, canvas.width, canvas.width);

				shownImageBitmap = imageBitmap;
			}),
			// Effect.tap(openNewTabWithPunklineImage),
		)
	}
</script>

<div class="flex-1 flex flex-col justify-center items-center gap-4">
	<h1
		class="font-mono text-4xl bg-gradient-to-r from-[#DC1FFF] via-[#03E1FF] to-[#00FFA3] inline-block text-transparent bg-clip-text"
	>
		PUNKLINES
	</h1>
	<h2
		class="font-mono text-xl bg-gradient-to-r from-[#DC1FFF] via-[#03E1FF] to-[#00FFA3] inline-block text-transparent bg-clip-text"
	>
		Wallpaper Generator
	</h2>
	{#if isLoading}
		<div class="flex justify-center items-center w-[270px] h-[480px]" style:background>
			<span class="loading loading-spinner loading-lg" />
		</div>
	{:else}
		<canvas
			class="rounded-md"
			width=270
			height=480
			style:background
			bind:this={canvas}
		/>
	{/if}
	<div class="flex flex-col gap-4">
		<input
			bind:value={punklinesNumberInput}
			type="number"
			placeholder="PUNKLINES no."
			class="font-mono input input bordered input-accent"
			on:input={debouncedInputChange}
		/>
		<button
			class="font-mono btn btn-accent"
			on:click={() => downloadImage()} 
		>
			GENERATE
		</button>
	</div>
</div>
