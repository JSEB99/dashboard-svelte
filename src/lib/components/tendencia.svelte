<script lang="ts">
	import Chart from 'chart.js/auto';
	import 'chartjs-adapter-date-fns';
	import { onMount } from 'svelte';

	export let transposedData: any;
	let transposedData2: any;
	$: {
		transposedData2 = transposedData;
	}

	let ctx: CanvasRenderingContext2D;
	let canvas: HTMLCanvasElement;
	function createChart() {
		//const transposedData = getDummyData();
		ctx = canvas.getContext('2d')!;
		new Chart(ctx, {
			type: 'line',
			data: {
				datasets: transposedData2 // Utiliza la variable aquí
			},
			options: {
				plugins: {
					legend: {
						labels: {
							color: 'rgb(255, 255, 255)'
						}
					}
				},
				scales: {
					x: {
						type: 'time',
						time: {
							unit: 'hour',
							displayFormats: {
								minute: 'HH:mm'
							}
						},
						title: {
							display: true,
							text: 'Tiempo',
							color: 'rgb(0, 255, 0)'
						},
						ticks: {
							color: 'rgb(0, 255, 0)'
						}
					},
					y: {
						title: {
							display: true,
							text: 'Valor',
							color: 'rgb(0, 255, 0)'
						},
						ticks: {
							color: 'rgb(0, 255, 0)'
						}
					}
				}
			}
		});
	}

	onMount(createChart);
</script>

<div class="chart-container">
	<canvas bind:this={canvas} />
</div>

<style>
	:global(body) {
		padding: 0;
	}
	div {
		width: 600px;
		height: 300px;
	}
</style>
