<script lang="ts">
	import { onMount } from 'svelte';
	import type { DatasetInfo } from '$lib/types/dataset_info';
	import { LogarithmicScale } from 'chart.js';
	import Tendencia from '$lib/components/tendencia.svelte';
	import Piechart from '$lib/components/piechart.svelte';

	let datasetInfo: DatasetInfo;

	let valorSelect: string = '13';

	let tendenciaData: Promise<any> = getDummyData(valorSelect);

	async function getDummyData(fechaSesion: string): Promise<any> {
		const API_URL = `http://127.0.0.1:8000/eventos/${fechaSesion}`;

		const response = await fetch(API_URL);
		const jsonData = await response.json(); // Espera la respuesta y el JSON

		const transposedData = jsonData.columns.map((column: string, columnIndex: number) => ({
			label: column,
			data: jsonData.data.map((row: any[], rowIndex: number) => ({
				x: jsonData.index[rowIndex],
				y: row[columnIndex]
			})),
			hoverBackgroundColor: 'rgb(0, 255, 0)'
		}));

		console.log(jsonData);
		console.log(transposedData);
		//tendenciaData = transposedData;
		return transposedData;
	}

	async function updateData(): Promise<void> {
		tendenciaData = getDummyData(valorSelect);
		pieChartData = getPieChartData(valorSelect);
		await obtenerDatos();
		await obtenerMenorTiempo();
		console.log(valorSelect);
	}

	let pieChartData = getPieChartData(valorSelect);

	async function getPieChartData(fechaSesion: string): Promise<any> {
		const API_URL = `http://127.0.0.1:8000/asistencia/${fechaSesion}`;

		const response = await fetch(API_URL);
		const jsonData = await response.json(); // Espera la respuesta y el JSON

		const dataArray = Object.values(jsonData);

		console.log(dataArray);

		return dataArray;
	}

	let total: number = 0;
	let promedio: number = 0;
	let std: number = 0;

	async function obtenerDatos() {
		const response = await fetch(`http://127.0.0.1:8000/analisis/${valorSelect}`);
		const jsonData = await response.json();

		total = jsonData.total;
		promedio = jsonData.promedio;
		std = jsonData.std;
	}
	onMount(obtenerDatos);

	let nombreCompleto: string = '';
	let correo: string = '';
	let duracion: number = 0;

	async function obtenerMenorTiempo() {
		const response = await fetch(`http://127.0.0.1:8000/menor_tiempo/${valorSelect}`);
		const jsonData = await response.json();

		const id = Object.keys(jsonData)[0]; // Obtener la clave (por ejemplo, "38")
		const item = jsonData[id];
		nombreCompleto = item.nombre_completo;
		correo = item.correo;
		duracion = item.duración;

		// Puedes hacer lo que necesites con estos valores aquí
		console.log('ID:', id);
		console.log('Nombre completo:', nombreCompleto);
		console.log('Correo:', correo);
		console.log('Duración:', duracion);
	}
	onMount(obtenerMenorTiempo);
</script>

<nav class="navbar">
	<div class="navbar-title">CyberSecurity Event Analytics</div>
	<div class="event-selector">
		<label for="event-selector" id="label-selector"
			>Selecciona un Evento <br />
			<span class="label-span">Dando Click</span></label
		>
		<select
			id="evento-selector"
			bind:value={valorSelect}
			on:change={() => {
				void updateData();
			}}
		>
			<option value="06">Evento 06</option>
			<option value="08">Evento 08</option>
			<option value="13">Evento 13</option>
			<option value="15">Evento 15</option>
		</select>
	</div>
</nav>

<div class="card-grid">
	<div class="card card1">
		<span class="card1-num">{total}</span>
		<span class="card1-text">Número de Asistentes</span>
	</div>
	<div class="card card2">
		<span class="card1-num">{promedio}</span>
		<span class="card1-text">Duración promedio</span>
	</div>
	<div class="card card3">
		<span class="card1-num">{std}</span>
		<span class="card1-text">Desviación estandar de la duración</span>
	</div>
	<div class="card card4">
		<span class="card2-num"
			>{nombreCompleto}<br />{correo}<br />Duración:
			<strong class="card2-duration">{duracion}</strong> min.</span
		>
		<span class="card1-text">Menor Tiempo</span>
	</div>
	<div class="card card5">
		{#await tendenciaData}
			<p>Loading...</p>
		{:then tendenciaData}
			<!-- <p>Listos:</p> -->
			<Tendencia transposedData={tendenciaData} />
		{/await}
	</div>
	<div class="card card6">
		{#await pieChartData}
			<p>Loading...</p>
		{:then pieChartData}
			<!-- <p>Listos:</p> -->
			<Piechart dataArray={pieChartData} />
		{/await}
	</div>
	<!-- Agrega más tarjetas si es necesario -->
</div>

<style>
	/* Estilos para la barra de navegación */
	.navbar {
		display: flex;
		justify-content: space-between;
		background-color: rgba(0, 0, 0, 0.7);
		color: white;
		padding: 10px;
		margin: 0 10px;
		margin-top: 10px;
		border-radius: 10px;
	}

	.navbar-title {
		font-size: 24px;
		font-weight: 700;
		font-style: italic;
		color: rgb(0, 255, 0);
	}

	.event-selector {
		display: flex;
		align-items: center;
	}

	#evento-selector {
		background-color: rgb(8, 8, 8);
		padding: 10px 50px;
		border-radius: 5px;
		border-color: rgb(0, 255, 0);
		color: rgb(0, 255, 0);
		font-size: 15px;
		font-weight: 600;
		appearance: none;
		margin-left: 30px;
	}
	#label-selector {
		font-weight: 600;
	}

	.label-span {
		font-weight: 400;
		font-size: 10px;
	}

	/* Estilos para la cuadrícula de tarjetas */
	.card-grid {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		grid-auto-rows: minmax(
			190px,
			auto
		); /* Las filas tienen altura mínima de 100px y crecerán según el contenido */
		gap: 10px;
		padding: 10px;
	}

	.card {
		background-color: rgba(0, 0, 0, 0.7);
		padding: 20px;
		border-radius: 10px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-around;
		text-align: center;
	}
	.card1-num {
		color: white;
		font-size: 3em;
		margin-top: 10px;
	}
	.card2-num {
		color: white;
		font-size: 1em;
		line-height: 2;
	}
	.card2-duration {
		font-size: 1.5em;
	}
	.card1-text {
		color: rgb(0, 255, 0);
	}

	/* Estilos para las tarjetas específicas */
	.card1 {
		grid-area: 1 / 1 / 1 / 1;
	}

	.card2 {
		grid-column: 2 / span 1;
		grid-row: 1 / span 1;
	}

	.card3 {
		grid-column: 3 / span 1;
		grid-row: 1 / span 1;
	}

	.card4 {
		grid-column: 4 / span 1;
		grid-row: 1 / span 1;
	}

	.card5 {
		grid-area: 2 / 1 / 4 / 3; /* Ocupa dos columnas */
	}

	.card6 {
		grid-column: span 2; /* Ocupa dos columnas */
		grid-row: span 2; /* Ocupa dos filas */
	}

	/* Puedes agregar más estilos para tarjetas adicionales si es necesario */
	.card5 p {
		margin: 0;
	}
</style>
