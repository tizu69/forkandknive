<script lang="ts">
	import { onDestroy, onMount } from 'svelte';

	export let img: ImageData;
	export let overrides: Record<string, string>;

	let canvas: HTMLCanvasElement;
	let interval: NodeJS.Timeout;

	onMount(
		() =>
			(interval = setInterval(() => {
				const ctx = canvas.getContext('2d');
				if (!ctx) throw new Error('stupid canvas');

				const width = (canvas.width = img.width);
				const height = (canvas.height = img.height);
				ctx.putImageData(img, 0, 0);

				const imageData = ctx.getImageData(0, 0, width, height);
				const data = imageData.data;

				for (let i = 0; i < data.length; i += 4) {
					const r = data[i];
					const g = data[i + 1];
					const b = data[i + 2];
					const a = data[i + 3];
					const key = `rgba(${r},${g},${b},${a})`;

					for (const [set, to] of Object.entries(overrides))
						if (set == key) {
							const [newR, newG, newB, newA] = to.replace('rgba(', '').replace(')', '').split(',').map(Number);
							data[i] = newR;
							data[i + 1] = newG;
							data[i + 2] = newB;
							data[i + 3] = newA * 255;
						}
				}

				ctx.putImageData(imageData, 0, 0);
			}, 500))
	);
	onDestroy(() => clearInterval(interval));
</script>

<canvas bind:this={canvas} {...$$restProps} />
