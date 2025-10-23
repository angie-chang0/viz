<script lang="ts">
	import AQIChart from '$lib/AQIChart.svelte';
	import AQIChartPart2 from '$lib/AQIChartPart2.svelte';
	import * as d3 from 'd3';

	const datasets = {
		avalon: 'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BAvalon%5D_daily-avg.csv',
		glassport_high_street:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BGlassport%20High%20Street%5D_daily-avg.csv',
		lawrenceville:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BLawrenceville%5D_daily-avg.csv',
		liberty_sahs:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BLiberty%20(SAHS)%5D_daily-avg.csv',
		manchester:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BManchester%5D_daily-avg.csv',
		north_braddock:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BNorth%20Braddock%5D_daily-avg.csv',
		parkway_east_near_road:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BParkway%20East%20(Near%20Road)%5D_daily-avg.csv',
		usa_pennsylvania_pittsburgh:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BUSA-Pennsylvania-Pittsburgh%5D_daily-avg.csv'
	};

	let selectedDatasetKey = $state('avalon');
	let showRawData = $state(false);
	
	// Part 2 controls
	let selectedDatasetKey2 = $state('avalon');
	let showRawData2 = $state(false);

	const stationOptions = [
		{ key: 'lawrenceville', name: 'Lawrenceville', count: 2419 },
		{ key: 'usa_pennsylvania_pittsburgh', name: 'USA-Pennsylvania-Pittsburgh', count: 2419 },
		{ key: 'avalon', name: 'Avalon', count: 2419 },
		{ key: 'glassport_high_street', name: 'Glassport High Street', count: 2419 },
		{ key: 'liberty_sahs', name: 'Liberty (SAHS)', count: 2419 },
		{ key: 'manchester', name: 'Manchester', count: 2419 },
		{ key: 'north_braddock', name: 'North Braddock', count: 2419 },
		{ key: 'parkway_east_near_road', name: 'Parkway East (Near Road)', count: 2419 }
	];

	const data = $derived.by(async () => {
		const url = datasets[selectedDatasetKey as keyof typeof datasets];
		if (!url) return [];
		
		const data = await d3.csv(url, (d: any) => ({
			city: d.City,
			country: d.Country,
			mainPollutant: d['Main pollutant'],
			pm25: +d['PM2.5'],
			state: d.State,
			stationName: d['Station name'],
			timestamp: new Date(d['Timestamp(UTC)']),
			usAqi: +d['US AQI']
		}));
		return data;
	});

	const data2 = $derived.by(async () => {
		const url = datasets[selectedDatasetKey2 as keyof typeof datasets];
		if (!url) return [];
		
		const data = await d3.csv(url, (d: any) => ({
			city: d.City,
			country: d.Country,
			mainPollutant: d['Main pollutant'],
			pm25: +d['PM2.5'],
			state: d.State,
			stationName: d['Station name'],
			timestamp: new Date(d['Timestamp(UTC)']),
			usAqi: +d['US AQI']
		}));
		return data;
	});
</script>


{#await data}
	<p>Loading data, one sec...</p>
{:then data}
<div class="chart-wrapper">
	<div class="controls-inline">
		<div class="control-group">
			<label for="station-select">Station:</label>
			<select id="station-select" bind:value={selectedDatasetKey}>
				{#each stationOptions as option}
					<option value={option.key}>{option.name} ({option.count})</option>
				{/each}
			</select>
		</div>
		
		<div class="control-group">
			<label>
				<input type="checkbox" bind:checked={showRawData} />
				Show Raw Data
			</label>
		</div>
		
		<p>Number of records: {stationOptions.find(o => o.key === selectedDatasetKey)?.count || 0}</p>
	</div>
	
	<h2>Part 1: AQI Chart</h2>
	<AQIChart {data} {showRawData} />
	
	<!-- Part 2 -->
	<div class="controls-inline">
		<div class="control-group">
			<label for="station-select-2">Station:</label>
			<select id="station-select-2" bind:value={selectedDatasetKey2}>
				{#each stationOptions as option}
					<option value={option.key}>{option.name} ({option.count})</option>
				{/each}
			</select>
		</div>
		
		<div class="control-group">
			<label>
				<input type="checkbox" bind:checked={showRawData2} />
				Show Raw Data
			</label>
		</div>
		
		<p>Number of records: {stationOptions.find(o => o.key === selectedDatasetKey2)?.count || 0}</p>
	</div>
	
	<h2>Part 2</h2>
	{#await data2}
		<p>Loading Part 2 data...</p>
	{:then data2}
		<AQIChartPart2 data={data2} showRawData={showRawData2} />
	{:catch error}
		<p>Error loading Part 2 data: {error.message}</p>
	{/await}
</div>
{:catch error}
	<!-- promise was rejected -->
	<p>Something went wrong: {error.message}</p>
{/await}

<style>
	* {
		font-family: sans-serif;
	}

	.chart-wrapper {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 100%;
	}

	.controls-inline {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		gap: 10px;
		margin-bottom: 20px;
		padding: 10px;
		width: 1200px;
	}

	.control-group {
		display: flex;
		align-items: center;
		gap: 5px;
	}

	.control-group label {
		font-weight: normal;
		margin-right: 5px;
	}

	select {
		padding: 5px;
		border: 1px solid #ccc;
		border-radius: 3px;
		font-size: 14px;
	}

	input[type="checkbox"] {
		margin-right: 5px;
	}

	p {
		margin: 0;
		font-size: 14px;
		font-weight: normal;
	}
</style>
