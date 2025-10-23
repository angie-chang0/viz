<script lang="ts">
	import * as d3 from 'd3';

	interface Item {
		city: string;
		country: string;
		mainPollutant: string;
		pm25: number;
		state: string;
		stationName: string;
		timestamp: Date;
		usAqi: number;
	}

	interface MonthlyData {
		month: Date;
		average: number;
		p10: number;
		p90: number;
		raw: Item[];
	}

	const Levels = [
		{"name":"Good","min":0,"max":50,"color":"#9cd84e"},
		{"name":"Moderate","min":50,"max":100,"color":"#facf39"},
		{"name":"Unhealthy for Sensitive Groups","min":100,"max":150,"color":"#f99049"},
		{"name":"Unhealthy","min":150,"max":200,"color":"#f65e5f"},
		{"name":"Very Unhealthy","min":200,"max":300,"color":"#a070b6"},
		{"name":"Hazardous","min":300,"color":"#a06a7b"}
	];

	let chartWidth = $state(1200);
	let chartHeight = $state(600);
	let chartMargins = $state({ top:20, right:30, bottom:40, left: 60 });

	const { data, showRawData = false }: { data: Item[]; showRawData?: boolean } = $props();

	let processedData = $derived(
		(() => {
			if (!data || data.length === 0) return [];

			const monthlyGroups = d3.group(data, d => {
				const date = new Date(d.timestamp);
				return new Date(date.getFullYear(), date.getMonth(), 15);
			});

			const monthlyData = Array.from(monthlyGroups.entries()).map(([month, values]) => {
				const aqiValues = values.map(d => d.usAqi).sort((a, b) => a - b);
				
				return {
					month: month,
					average: d3.mean(aqiValues) || 0,
					p10: d3.quantile(aqiValues, 0.1) || 0,
					p90: d3.quantile(aqiValues, 0.9) || 0,
					raw: values
				};
			}).sort((a, b) => a.month.getTime() - b.month.getTime());

			return monthlyData;
		})() as MonthlyData[]
	);

	let timeScale = $derived(
		d3.scaleTime()
			.domain([
				d3.min(processedData, d => d.month) ?? new Date(),
				d3.max(processedData, d => d.month) ?? new Date()
			])
			.range([chartMargins.left,chartWidth - chartMargins.right])
	);

	let aqiScale = $derived(
		d3.scaleLinear()
			.domain([0, 160])
			.range([chartHeight - chartMargins.bottom, chartMargins.top])
	);

	let averageLinePath = $derived(
		d3.line<MonthlyData>()
			.x(d => timeScale(d.month))
			.y(d => aqiScale(d.average))
			.curve(d3.curveLinear)(processedData)
	);

	let percentileAreaPath = $derived(
		d3.area<MonthlyData>()
			.x(d => timeScale(d.month))
			.y0(d => aqiScale(d.p90))
			.y1(d => aqiScale(d.p10))
			.curve(d3.curveLinear)(processedData)
	);

	let baseTimeScale = $derived(timeScale);
	let baseAqiScale = $derived(aqiScale);

	// Curr scales
	let currentTimeScale = $derived(timeScale);
	let currentAqiScale = $derived(aqiScale);

	let xAxisGenerator = $derived(d3.axisBottom(currentTimeScale));
	let yAxisGenerator = $derived(d3.axisLeft(currentAqiScale));

	let xAxisRef: SVGGElement;
	let yAxisRef: SVGGElement;
	let svgRef: SVGSVGElement;
	
	// Hover state
	let mouseX = $state(0);
	let mouseY = $state(0);
	let hoveredData: MonthlyData | null = $state(null);
	let showTooltip = $state(false);

	// Find nearest data point
	function findNearestDataPoint(mouseX: number) {
		if (!processedData || processedData.length === 0) return null;
		
		let closestPoint: MonthlyData = processedData[0];
		let closestDistance = Infinity;

		processedData.forEach((d) => {
			const x = currentTimeScale(d.month);
			const distance = Math.abs(x - mouseX);
			if (distance < closestDistance) {
				closestPoint = d;
				closestDistance = distance;
			}
		});
		
		return closestPoint;
	}

    //zoom
	$effect(() => {
		if (!svgRef) return;

		const zoomed = (event: any) => {
			currentTimeScale = event.transform.rescaleX(baseTimeScale);
			currentAqiScale = event.transform.rescaleY(baseAqiScale);

			if (xAxisRef) {
				d3.select(xAxisRef).call(d3.axisBottom(currentTimeScale));
			}
			if (yAxisRef) {
				d3.select(yAxisRef).call(d3.axisLeft(currentAqiScale));
			}
		};

		const zoomBehavior = d3
			.zoom<SVGSVGElement, unknown>()
			.scaleExtent([1, 10])
			.translateExtent([
				[chartMargins.left, chartMargins.top],
				[chartWidth - chartMargins.right, chartHeight - chartMargins.bottom]
			])
			.on('zoom', zoomed);

		d3.select(svgRef).call(zoomBehavior);
	});

	$effect(() => {
		if (xAxisRef && data.length > 0) {
			d3.select(xAxisRef).call(xAxisGenerator);
		}
		if (yAxisRef && data.length > 0) {
			d3.select(yAxisRef).call(yAxisGenerator);
		}
	});
</script>

<div class="chart-container" style="position: relative;">
	<svg width={chartWidth} height={chartHeight} bind:this={svgRef}
		role="img"
		onmousemove={(e) => {
			mouseX = e.offsetX;
			mouseY = e.offsetY;
			hoveredData = findNearestDataPoint(mouseX);
			showTooltip = true;
		}}
		onmouseleave={() => {
			showTooltip = false;
			hoveredData = null;
		}}>
		<defs>
			<clipPath id="clip">
				<rect x={chartMargins.left} y={chartMargins.top} 
					width={chartWidth - chartMargins.left - chartMargins.right} 
					height={chartHeight - chartMargins.top - chartMargins.bottom} />
			</clipPath>
		</defs>

		<g clip-path="url(#clip)">
			{#each Levels as level}
				{@const yMax = currentAqiScale(level.max) || 0}
				{@const yMin = currentAqiScale(level.min) || 0}
				{@const yHazardous = currentAqiScale(300) || 0}
				<rect
					x={chartMargins.left}
					y={level.name === 'Hazardous' ? chartMargins.top : yMax}
					width={chartWidth - chartMargins.left - chartMargins.right}
					height={level.name === 'Hazardous' ? yHazardous - chartMargins.top : yMin - yMax}
					fill={level.color}
					opacity="0.4"
				/>
			{/each}

			{#if processedData.length > 0}
				{@const areaPath = d3.area<MonthlyData>()
					.x(d => currentTimeScale(d.month))
					.y0(d => currentAqiScale(d.p90))
					.y1(d => currentAqiScale(d.p10))
					.curve(d3.curveLinear)(processedData)}
				<path
					d={areaPath}
					fill="#666"
					opacity="0.3"
				/>
			{/if}

			{#if processedData.length > 0}
				{@const linePath = d3.line<MonthlyData>()
					.x(d => currentTimeScale(d.month))
					.y(d => currentAqiScale(d.average))
					.curve(d3.curveLinear)(processedData)}
				<path
					d={linePath}
					fill="none"
					stroke="#000"
					stroke-width="2"
				/>
			{/if}

			{#if showRawData && data.length > 0}
				{#each data as item}
					<circle
						cx={currentTimeScale(item.timestamp)}
						cy={currentAqiScale(item.usAqi)}
						r="1.5"
						fill="#000"
						opacity="0.8"
					/>
				{/each}
			{/if}

			{#if hoveredData && showTooltip}
				<circle
					cx={currentTimeScale(hoveredData.month)}
					cy={currentAqiScale(hoveredData.average)}
					r="6"
					fill="red"
					stroke="white"
					stroke-width="2"
				/>
			{/if}
		</g>

		<g id="x-axis" transform="translate(0,{chartHeight - chartMargins.bottom})" bind:this={xAxisRef}></g>
		<g id="y-axis" transform="translate({chartMargins.left},0)" bind:this={yAxisRef}></g>
	</svg>

	{#if showTooltip && hoveredData}
		<div
			style="
				position: absolute;
				left: {mouseX + 10}px;
				top: {mouseY - 20}px;
				pointer-events: none;
				z-index: 1000;
				background: white;
				border: 2px solid #ccc;
				border-radius: 4px;
				padding: 8px 12px;
				box-shadow: 0 2px 8px rgba(0,0,0,0.15);
			"
			class="text-sm"
		>
			<div class="font-semibold">Mean AQI: {hoveredData.average.toFixed(1)}</div>
			<div>Date: {hoveredData.month.toLocaleDateString()}</div>
			<div>Range: {hoveredData.p10.toFixed(1)} - {hoveredData.p90.toFixed(1)}</div>
		</div>
	{/if}
</div>

<style>
	svg {
		font-family: sans-serif;
		font-size: 12px;
	}

	.chart-container {
		display: flex;
		justify-content: center;
	}
</style>
