<script>
    import { Plot, GridX, BarX, AxisX, AxisY, Text, HTMLTooltip, Pointer } from 'svelteplot';
    import { format } from "d3-format";
    import { timeParse, timeFormat} from "d3-time-format"
    import { extent, min, max, sum, sort, ascending, descending } from "d3-array"

	let { 
        data, 
        variant = "simple",
        xKey, 
        yKey,
        zKey, 
        xDomain = "auto",
        xFormat,
        xFormatDate,
        xAxisTicks,
        yDomain,
        yFormat,
        yFormatDate,
        zFormat,
        zFormatDate,
		ySort,
        zSortKey, 
        dataLabels,
        tooltip,
        height,
        seriesHeight = 30,
        margin = {top: 20, bottom: 40, right: 80}, 
        colours = ['#206095','#A8BD3A','#871A5B','#F66068']
    } = $props();

    let domainX = $derived.by(() => {
        if(xDomain == "auto" && variant != "stacked"){
            if(min(data.map((d) => d[xKey])) < 0){
                return extent(data.map((d) => d[xKey]))
            } else{
                return [0, max(data.map((d) => d[xKey]))]
            }
        } else if(xDomain == "auto" && variant == "stacked"){
            let maxes = []
            let filteredData;
            let yKeys = [...new Set(data.map((d) => d[yKey]))]
            yKeys.forEach((category) => {
                filteredData = data.filter((d) => d[yKey] == category)
                maxes.push(sum(filteredData.map((d) => d[xKey])))
            })
            return [0, max(maxes)]
        }
    })

    let derivedHeight = $derived.by(() => {
        if(height){
            return height
        } else{
            if(variant == "clustered"){
                return seriesHeight * ([...new Set(data.map((d) => d[yKey]))].length * [...new Set(data.map((d) => d[zKey]))].length)
            } else{
                return seriesHeight * [...new Set(data.map((d) => d[yKey]))].length
            }
        }
    })

    let domainY = $derived.by(() => {
        if(ySort == "ascending" && !zSortKey && variant == "stacked"){
            let sortedData = [];
            [...new Set(data.map((d) => d[yKey]))].forEach((category) => {
                sortedData.push({
                    yKey: category,
                    sum: sum(data.filter((d) => d[yKey] == category).map((d) => d[xKey]))
                })
            })
            sortedData = sortedData.sort((a, b) => ascending(a.sum, b.sum))
            return [...new Set(sortedData.map((d) => d.yKey))]
        } else if(ySort == "descending" && !zSortKey && variant == "stacked"){
            let sortedData = [];
            [...new Set(data.map((d) => d[yKey]))].forEach((category) => {
                sortedData.push({
                    yKey: category,
                    sum: sum(data.filter((d) => d[yKey] == category).map((d) => d[xKey]))
                })
            })
            sortedData = sortedData.sort((a, b) => descending(a.sum, b.sum))
            return [...new Set(sortedData.map((d) => d.yKey))]
        } else if(ySort == "ascending" && !zSortKey){
            let sortedData = data.sort((a, b) => ascending(a[xKey], b[xKey]))
            return [...new Set(sortedData.map((d) => d[yKey]))]
        } else if(ySort == "descending" && !zSortKey){
            let sortedData = data.sort((a, b) => descending(a[xKey], b[xKey]))
            return [...new Set(sortedData.map((d) => d[yKey]))]
        }
        else if(ySort == "ascending" && zSortKey){
            let sortedData = data.filter((d) => d[zKey] == zSortKey).sort((a, b) => ascending(a[xKey], b[xKey]))
            return [...new Set(sortedData.map((d) => d[yKey]))]
        } else if(ySort == "descending" && zSortKey){
            let sortedData = data.filter((d) => d[zKey] == zSortKey).sort((a, b) => descending(a[xKey], b[xKey]))
            return [...new Set(sortedData.map((d) => d[yKey]))]
        } else{
            return [...new Set(data.map((d) => d[yKey]))]
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
    height = {derivedHeight ? derivedHeight : height} 
    y={{ domain: variant == "clustered" ? null : domainY, tickSpacing: 10, label:"", tickFormat: (d) => variant == "clustered" ? "" : yFormatDate ? timeFormat(yFormat)(timeParse(yFormatDate)(d)) : yFormat ? format(yFormat)(d) : d}} 
    x={{ domain: domainX, label:""}}
    color={{ legend: variant == "clustered" || variant == "stacked" ? true : false}}
    fy={{
        axis: 'left',
        anchor: 'left',
        domain: variant == "clustered" ? domainY : null,
        axisOptions: {
            dx: -margin.right - 10
        }
    }}
    fx={{
        tickFormat: (d) => zFormatDate ? timeFormat(zFormat)(timeParse(zFormatDate)(d)) : zFormat ? format(zFormat)(d) : d
    }}
>
    <AxisX
       tickCount={xAxisTicks}
       tickFormat ={ (d) => xFormatDate ? timeFormat(xFormat)(timeParse(xFormatDate)(d)) : xFormat ? format(yFormat)(d) : d }
    />
	<GridX stroke="#D9D9D9"/>
    <BarX 
        data={data}
        x={xKey} 
        y={variant == "clustered" ? zKey : yKey}
        fy={variant == "clustered" ? yKey : null}
        fx={variant == "small-multiple" ? zKey : null}
        order={variant == "clustered" ? 'z' : null}
        sort={!ySort ? false : ySort == "ascending" ? { channel: 'x' } : { channel: '-x' }}
        fill={variant == "stacked" || variant == "clustered" ? zKey : true}
    />

    {#if dataLabels}
        <Text
            data={data}
            x={xKey} 
            y={variant == "clustered" ? zKey : yKey}
            fy={variant == "clustered" ? yKey : null}
            fx={variant == "small-multiple" ? zKey : null}
            textAnchor={(d) => d[xKey] < domainX[1]*0.2 ? "start" : "end"}
            dx={(d) => d[xKey] < domainX[1]*0.2 ? 4 : -4}
            text={(d) => dataLabels.format ? format(dataLabels.format)(d[xKey]) : d[xKey]}
            textClass="dataLabel"
            fill={(d) => d[xKey] < domainX[1]*0.2 ? "#414042" : "#FFFFFF"}
        />
    {/if}
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
    :global(.dataLabel){
        font-weight: 600 !important;
        font-size: 14px !important;
    }
    :global(.item){
        font-size: 16px !important;
    }
    :global(.item rect){
        ry: 12;
    }
    .tooltip {
        background: white;
        border-color: #414042;
        font-size: 14px;
        padding: 1ex 1em;
        border-radius: 3px;
        line-height: 1.2;
        /* box-shadow:
            rgba(50, 50, 93, 0.25) 0px 2px 5px -1px,
            rgba(0, 0, 0, 0.3) 0px 1px 3px -1px; */
    }
</style>