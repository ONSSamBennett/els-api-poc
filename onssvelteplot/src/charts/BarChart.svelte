<script>
    import { Plot, GridX, BarX, stackY } from 'svelteplot';
    import { min, max, range, extent, map } from 'd3-array';
	import { tweened } from 'svelte/motion';
	import { derived } from 'svelte/store';
	import { cubicOut } from 'svelte/easing';
	import { onMount } from 'svelte';
    import { sort, ascending, descending } from "d3-array";

	let { 
        data, 
        variant = "simple",
        xKey, 
        yKey,
        zKey, 
        xDomain,
        yDomain,
		ySort = "ascending", 
        height = 350, 
        margin = {top: 20, bottom: 40, right: 80}, 
        colours = ['#206095','#A8BD3A','#871A5B','#F66068']
    } = $props();

    let yTickFormat = $derived.by(() => {
        if(variant == "clustered"){
            return ""
        } else{
            return tickFormat
        }
    })

    // let chartData = $derived.by(() => {
    //     if(ySort == "ascending"){
    //         return sort(data, (a, b) => ascending(a[xKey], b[xKey]))
    //     } else if(ySort == "descending"){
    //         return sort(data, (a, b) => descending(a[xKey], b[xKey]))
    //     } else{
    //         return data
    //     }
    // })

    // let chartYDomain = $derived(!yDomain ? [...new Set(chartData.map((d) => d[yKey]))] : yDomain)

</script>

<Plot 
    marginRight={margin.right} 
    marginTop={margin.top} 
    marginBottom={margin.bottom} 
    {height} 
    y={{ domain: yDomain, label:"", tickFormat: (d) => variant == "clustered" ? "" : d}} 
    x={{ domain: xDomain, label:"" }}
    color={{ legend: variant == "simple" ? false : true }}
    fy={{
        axis: 'left',
        anchor: 'left',
        axisOptions: {
            dx: -margin.right - 10
        }
    }}
>
	<GridX stroke="#D9D9D9"/>
    <BarX 
        data={data}
        x={xKey} 
        y={variant == "clustered" ? zKey : yKey}
        fy={variant == "clustered" ? yKey : null}
        sort={!ySort ? false : ySort == "ascending" ? { channel: 'x' } : { channel: '-x' }}
        fill={variant == "stacked" || variant == "clustered" ? zKey : true}
    />
</Plot>

<style>
	:global(.tick line, .grid-x line, .grid-y line){
		stroke: #D9D9D9 !important;
		stroke-width: 1px !important;
		stroke-opacity: 1 !important;
	}
	:global(text){
		font-family: 'OpenSans', 'Helvetica Neue', arial, sans-serif !important;
	}
	:global(.tick text){
		font-size: 16px !important;
	}
</style>