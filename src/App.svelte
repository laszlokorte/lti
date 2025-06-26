<script>
	let sampleCount = 25
	
	let selected = null
	let selectedOutput = null
	
	function selectOutput(v) {
		selected = null
		selectedOutput = v
	}
	
	function selectInput(v) {
		selected = v
		selectedOutput = null
	}
	
	$: inputSignal = (t) =>  (Math.sin(t*16*Math.PI)+3) * Math.exp(-0.5*Math.pow(t-0,2)/0.01) * 2
	$: impulseSignal = (t) => t<0? 0 : Math.exp(-t*5)
	
	$: inputSamples = Array(1000).fill(null).map((_,i,all) => i/all.length - 0.5).map((t) => inputSignal(t))
	$: inputSamplesLow = Array(sampleCount).fill(null).map((_,i,all) => i/all.length - 0.5).map((t) => inputSignal(t))
	$: impulseSamples = Array(1000).fill(null).map((_,i,all) => i/all.length).map((t) => impulseSignal(t))
	$: impulseSamplesLow = Array(sampleCount).fill(null).map((_,i,all) => i/all.length).map((t) => impulseSignal(t))

	$: outputSamplesLow = selected == null ? [] : Array(sampleCount).fill(null).map((_,i,all) => i/all.length).map((t) => impulseSignal(t) * inputSamplesLow[selected]*8/sampleCount)
	
	$: outputSamplesAllLow = Array(sampleCount).fill(null).map((_,i,all) => i/all.length).map((t) => inputSamplesLow.reduce((acc, s, k, all) => acc + impulseSignal(t-k/all.length) * s, 0)*8/sampleCount)
	
	
	$: outputSamplesReversedLow = selectedOutput == null ? [] : Array(sampleCount).fill(null).map((_,i,all) => i/all.length).map((t, i) => impulseSignal(t) * inputSamplesLow[selectedOutput-i]*8/sampleCount).filter((v) => !isNaN(v)).reduce((acc, v, i, all) => (acc.length ? [...acc, {offset: acc[acc.length-1].offset + acc[acc.length-1].val, val: v}] : [{offset: 0, val: v}]), [])
	
	$: console.log(outputSamplesReversedLow)
</script>

<style>
	article {
		margin: 0 auto;
		max-width: 60em;
	}
	
	h1, h2 {
		margin: 0;
	}
	
	dl {
		display: grid;
		gap: 1em;
		grid-template-columns: max-content max-content max-content;
		align-items: center;
	}
	
	dd {
		margin: 0;
	}
	
	input {
		margin: 0;
		padding: 0.3em;
	}
	
	
	.system {
		margin: 1em 0;
		display: flex;
		width: 100%;
		gap: 1em;
		align-content: center;
		justify-content: center;
		align-items: center;
		justify-items: center;
	}
	
	.graph {
		border: 1px solid gray;
		width: 100%;
		display: block;
	}
	
	.arrow {
		width: 5em;
		height: 3em;
	}
	
	figure {
		margin: 0;
		align-self: start;
	}
	
	figcaption {
		margin: 0;
		display: block;
		text-align: center;
	}
	
	.clickable {
		cursor: pointer;
	}
	
	.clickable:not(.selected):hover {
		opacity: 0.4;
	}
	
	.selected {
		fill: hsl(-90, 100%, 50%);
		opacity: 1;
	}

	@media (max-width: 35em) {
		.system {
			flex-direction: column;
			justify-content: stretch;
			justify-items: center;
		}

		.lti {
			width:  50%;
			margin: auto;
		}

		.arrow {
			transform: rotate(90deg);
		}

		figure {
			margin: 0;
			align-self: stretch;
		}
	}
</style>
<article>

	
<h1>
	LTI System impulse response
</h1>

<dl>
	<dt><label for="fmin">Sample Count</label></dt>
	<dd><input id="fmin" type="range" min="10" max={50} bind:value={sampleCount} on:input={() => {selected = null; selectedOutput = null}} /></dd>
	<dd><output>{sampleCount}</output></dd>


	<dt></dt>
	<dd><button disabled={selected==null && selectedOutput==null} on:click={() => {selected = null; selectedOutput = null}}>clear selection</button></dd>
	</dl>
	<p>
		Click onto one of the rectangles in the input signal to see how a single input value contributes to the output signal.
	</p>
	<p>
		Click onto one of the rectangle in the output signal to see from which values of the input signal it originates.
	</p>
	
<div class="system">

	<figure>
		<svg class="graph" viewBox="-110 -110 220 120">
			
			
			
		{#each inputSamplesLow as s,i}
			<rect shape-rendering="crispEdges" class="clickable" class:selected={i==selected} on:click={() => {selectInput(i)}} y="{s*-10}" x="{i/inputSamplesLow.length*200 - 100 - 200/inputSamplesLow.length/2}"  width="{200/inputSamplesLow.length}" height="{s*10}" opacity="0.2" stroke="white" vector-effect="non-scaling-stroke"></rect>
		{/each}
			
		{#if selectedOutput != null}
			{#each inputSamplesLow as s,i}
				{#if !(i > selectedOutput)}
					<rect shape-rendering="crispEdges" class="clickable" class:selected={i==selected} on:click={() => {selectInput(i)}} y="{s*-10}" x="{i/inputSamplesLow.length*200 - 100 - 200/inputSamplesLow.length/2}"  width="{200/inputSamplesLow.length}" height="{s*10}" fill="hsl({-90+360*(selectedOutput-i)/inputSamplesLow.length}, 100%, 50%)"></rect>
				{/if}
			{/each}
		{/if}
			
			<polyline fill="none" stroke="red" points={inputSamples.flatMap((v,i,all) => [i/all.length*200 - 100, v*-10])}></polyline>
			
			
			<g color="black" font-size="10" pointer-events="none">
				<line x1="-100" x2="100" y1="0" y2="0" stroke="currentColor" vector-effect="non-scaling-stroke" />
				<line y1="-100" y2="5" x1="0" x2="0" stroke="currentColor" vector-effect="non-scaling-stroke" />
				<polygon points="102 0 95 -3 95 3" fill="currentColor" />
				<polygon points="0 -102 -3 -95 3 -95" fill="currentColor" />
				<text x="95" y="-5" text-anchor="end">t</text>
				<text y="-95" x="5" text-anchor="start">f(t)</text>
			</g>
		</svg>
		<figcaption>
		input signal f(t)

	<input style="width: 100%;" type="range" min="0" max={sampleCount} bind:value={selected} on:input={() => {selectedOutput = null}} />

		</figcaption>
	</figure>
	<svg class="arrow" viewBox="0 -10 20 20">
		<polygon points="0 5 10 5 10 10 20 0 10 -10 10 -5 0 -5" />
	</svg>
	<figure>
	<svg class="graph lti" viewBox="-10 -110 220 120">		
		
		
		{#if selected != null || selectedOutput != null}
			{#each impulseSamplesLow as s,i}
				<rect shape-rendering="crispEdges" y="{s*-10*8}" x="{i/inputSamplesLow.length*200 - 200/inputSamplesLow.length/2}"  width="{200/inputSamplesLow.length}" height="{s*10*8}" fill="hsl({-90+360*i/impulseSamplesLow.length}, 100%, 50%)"></rect>
			{/each}
		{/if}
		<polyline fill="none" stroke="blue" points={impulseSamples.flatMap((v,i,all) => [i/all.length*200, v*-80])}></polyline>
		
		
		<g color="black" font-size="10" pointer-events="none">
			<line x1="-5" x2="200" y1="0" y2="0" stroke="currentColor" vector-effect="non-scaling-stroke" />
			<line y1="-100" y2="5" x1="0" x2="0" stroke="currentColor" vector-effect="non-scaling-stroke" />
			<polygon points="202 0 195 -3 195 3" fill="currentColor" />
			<polygon points="0 -102 -3 -95 3 -95" fill="currentColor" />
			<text x="195" y="-5" text-anchor="end">t</text>
			<text y="-95" x="5" text-anchor="start">h(t)</text>
			
			<line x1="-5" x2="5" y1="-80" y2="-80" stroke="currentColor" vector-effect="non-scaling-stroke" />
			<text y="-77" x="7" text-anchor="start">1</text>

		</g>	
		
	</svg>
		<figcaption>LTI system with impulse response h(t)</figcaption>
	</figure>
	
	<svg class="arrow" viewBox="0 -10 20 20">
		<polygon points="0 5 10 5 10 10 20 0 10 -10 10 -5 0 -5" />
	</svg>
	
	<figure>
	<svg class="graph" viewBox="-110 -110 220 120">
		
		
		{#each outputSamplesAllLow as s,i}
			<rect class="clickable" class:selected={i==selectedOutput} on:click={() => {selectOutput(i)}} y="{s*-10}" x="{i/inputSamplesLow.length*200 - 100 - 200/inputSamplesLow.length/2}"  width="{200/inputSamplesLow.length}" height="{s*10}" opacity="0.2" stroke="white" vector-effect="non-scaling-stroke" shape-rendering="crispEdges"></rect>
		{/each}
		
		<g pointer-events="none">
		{#each outputSamplesLow as s,i}
		<rect y="{s*-10}" x="{(selected+i)/inputSamplesLow.length*200 - 100 - 200/inputSamplesLow.length/2}"  width="{200/inputSamplesLow.length}" height="{s*10}" fill="hsl({-90+360*i/impulseSamplesLow.length}, 100%, 50%)" vector-effect="non-scaling-stroke" shape-rendering="crispEdges"></rect>
		{/each}
		</g>
		
		{#if selectedOutput != null}
		
		<g pointer-events="none">
			{#each outputSamplesReversedLow as {val, offset},i}
		<rect y="{(val+offset)*-10}" x="{(selectedOutput)/inputSamplesLow.length*200 - 100 - 200/inputSamplesLow.length/2}"  width="{200/inputSamplesLow.length}" height="{val*10}" fill="hsl({-90+360*i/impulseSamplesLow.length}, 100%, 50%)" shape-rendering="crispEdges" stroke="none" vector-effect="non-scaling-stroke"></rect>
		{/each}
		</g>
		
		{/if}
		
		
		<g color="black" font-size="10" pointer-events="none">
			<line x1="-100" x2="100" y1="0" y2="0" stroke="currentColor" vector-effect="non-scaling-stroke" />
			<line y1="-100" y2="5" x1="0" x2="0" stroke="currentColor" vector-effect="non-scaling-stroke" />
			<polygon points="102 0 95 -3 95 3" fill="currentColor" />
			<polygon points="0 -102 -3 -95 3 -95" fill="currentColor" />
			<text x="95" y="-5" text-anchor="end">t</text>
			<text y="-95" x="5" text-anchor="start">g(t)</text>
		</g>			
	</svg>
		<figcaption>
		output signal g(t)

	<input style="width: 100%;" type="range" min="0" max={sampleCount} bind:value={selectedOutput} on:input={() => {selected = null}} />
		</figcaption>
	</figure>
</div>
	
	<h2>
		Explanation
	</h2>
	
	<p>
		An linear and time-invariant system (LTI-system) transforms an input signal to an output signal. Linearity means that if the input is a sum of multiple individual signals, the output is the same as the sum of each individual transformed signal. Time-invariance means that if the input signal is shifted in time the output is the same as for the non-shifted input but shifted as well.
	</p>
	
	<p>
		These two properties allow to understand the systems behavior by understanding just it&apos;s response to a single impulse (very short rectangle). All other signals can the be thought of as a sum of shifted impulses.
	</p>
	
	<p>
		Above you can see how the input signal (red curve) can be decomposed into many rectangles. The blue curve shows the systems response to a single impulse. The output signal shows the systems output given the full input signal.
	</p>
	<p>
		By clicking onto the sampled rectangles of the input signal you can see how a single rectangle of the input contributes to the output by being replicated and scaled according to the impulse response. Clicking a rectangle in the output signal shows which rectangles from the input contributed to the selected output rectangle.
	</p>
	
</article>