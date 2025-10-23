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
	
	<h2>Part 2: Extreme AQI Values Explorer</h2>
	{#await data2}
		<p>Loading Part 2 data...</p>
	{:then data2}
		<AQIChartPart2 data={data2} showRawData={showRawData2} />
	{:catch error}
		<p>Error loading Part 2 data: {error.message}</p>
	{/await}
	
	<h2>Reflection</h2>
	
	<div class="reflection-section">
		<p class="question"><strong>How did you choose your particular visual encodings and interaction techniques? What alternatives did you consider and how did you arrive at your ultimate choices?</strong></p>
		
		<p class="answer">I choose to implement zoom and hover, so the user is able explore the data set more deeply. The user can zoom with the mouse wheel or by using fingers on a touchpad and dragging them out, or even double clicking. It's hard to see exactly what date and level it is, so zooming in and out allows more precision. In addition, when the user hovers over the line, a guiding data point is highlighted and the user can view the exact value, date, and range. I initially wanted to layer population data on top of the AQI data, but I realized it would have been hard to change the X scale to accurately show both the level and population. In addition, I was having difficulty sourcing that data reliably for each different location (Lawrenceville, Avalon, etc.). And I also wasn't too sure the visualization would look pleasing and might be confusing for the user to look at. I wanted the zooming to feel natural and easy to use, and it was definitely a bit technically difficult to implement for me in understanding how to use the rescale method. I also needed to calculate the nearest data point, by calculating the position of the mouse of the X scale and checking that with the closest monthly data point on the line.</p>
	</div>
	
	<div class="reflection-section">
		<p class="question"><strong>Describe the question that you are enabling a user to answer. The question should be compelling and the solution should be focused on helping users achieve their goals.</strong></p>
		
		<p class="answer">My second visualization takes part one and extends its concept to make it more exploratory and accessible. My guiding question was "What month had the most extreme levels?". To answer this question, the user should be able to closely examine the data points and zoom into the line to examine specific values at changes within the line. They should also be able to see the range that the mean was calculated over and the exact date correlated with that level. This is what drove me to specifically choose zoom and hover as the two primary interactions within this visualization. They are also still able to view the other locations through a toggle, and compare these exact values at key dates.</p>
	</div>
	
	<div class="reflection-section">
		<p class="question"><strong>Describe how the work was split among the team members. Include a commentary on the development process, including answers to the following questions: Roughly how much time did you spend developing your application (in people-hours)? What aspects took the most time? How did you use AI? Provide the prompts you used.</strong></p>
		
		<p class="answer">I did both part 1 and part 2 individually. For part 1, I watched many videos and articles that described how to implement visualizations in svelte and d3 
			(included below). I also incorporated AI into my process, 
			both for understanding how to get started and when I felt stuck. To understand how to get started, 
			I asked what certain functions did and what certain syntax meant. For when I got stuck or if something wasn't working, I copy pasted certain snippets, functions, and error messages to understand how to fix it. In addition, sometimes I would copy something from a tutorial and I wouldn't fully understand it so I would also ask AI or google/stackoverflow what was happening. To display both part 1 and part 2, on the same page, I was having trouble making them act separately, so I also used AI to help me load the states independently from each other in page.svelete. I left my prompts below. I took around 6 hours to fully complete and debug the first part and 4 hours on the second part. Learning syntax and debugging took most of the time, and I also took a lot of time learning how to include interactions for the second part. I initially got stuck on a separate idea that just wasn't working out so I had to backtrack and try to save my working code. Since I tried to implement everything in one sitting, I definitely wished I committed more versions of my code so I could go back to it when I messed up. For development, I first looked up how to use line charts, set up the scales, and create the header to switch between the different datasets, and also how to overlay a scatterplot over the raw data. One mistake I made was I used d3.curveMonotoneX by following a tutorial, and couldn't figure out my line graph was smoothed out, then I researched more about what the function actually does, realized my mistake then got replaced it to show actual data spikes. For part two, I had the same process as part 1, and built upon the chart by first copy pasting part 1 into a new file then adding the zoom and tooltip interactions to display data points. I completed most of this work in one sitting.</p>
	</div>
	
	<h4>AI Prompts Used:</h4>
	<ul>
		<li>Why are there gaps between the colors (I added an image)</li>
		<li>Why isn't this rendering (I included code for the dropdown menu, and asked why it wasn't displaying the dropdowns correctly)</li>
		<li>I want nearest point to mouse, why not working (copying in function)</li>
		<li>What does (error message) mean ( I prompted this many times)</li>
		<li>I want to deploy (project) (link) why 404</li>
		<li>New repo terminal instructions</li>
		<li>Why can't I move on y axis anymore (copy in zoom function)</li>
		<li>What is wrong with event state in this file, changing both graphs</li>
		<li>What does this do (I asked this a bunch of times with many functions based on tutorials)</li>
		<li>How to get mouse value</li>
		<li>Why is this not zooming in</li>
		<li>stop line curving (inserted picture)</li>
		<li>I dont understand why this works and why this is not working (tutorial code and my code)</li>
		<li>when hoever over the line, I want a dot to appear where the mouse is and to show eaxctly what the mean value is, why is it not appearing</li>
	</ul>
	
	<h4>Resources Used:</h4>
	<ul>
		<li><a href="https://medium.com/@stefano.agresti19/building-an-interactive-line-chart-using-svelte-and-d3-71841cf5703c" target="_blank">Building an Interactive Line Chart using Svelte and D3</a></li>
		<li><a href="https://www.youtube.com/watch?v=-THp2YVYEFc" target="_blank">D3.js Tutorial Video</a></li>
		<li><a href="https://d3-graph-gallery.com/graph/interactivity_zoom.html" target="_blank">Zoom Tutorial</a></li>
		<li><a href="https://d3-graph-gallery.com/graph/interactivity_tooltip.html" target="_blank">Tooltip/hover Tutorial</a></li>
		<li><a href="https://www.reddit.com/r/sveltejs/comments/126b993/svelte_d3_guides/" target="_blank">reddit post with tutorials</a></li>
		
	</ul>
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

	.reflection-section {
		margin-bottom: 30px;
		text-align: left;
		max-width: 1200px;
	}

	.question {
		font-size: 16px;
		margin-bottom: 15px;
		text-align: left;
	}

	.answer {
		text-align: left;
		line-height: 1.6;
		margin-bottom: 20px;
	}
</style>
