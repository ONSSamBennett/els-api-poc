<script>
  import { base } from "$app/paths";
  import {
    PhaseBanner,
    Header,
    Breadcrumb,
    Section,
    Footer,
    Select,
    Button,
    Divider,
    NavSections,
    NavSection,
    LazyLoad
  } from "@onsvisual/svelte-components";
  import Map from "$lib/viz/Map.svelte";
  import Bar from "$lib/viz/Bar.svelte";
  import Line from "$lib/viz/Line.svelte";
  import { fetchChartData } from "$lib/utils.js";
  import { BarChart } from "@onsvisual/onssvelteplot"

  export let data;

  let selected;
  let indicator;

  let barConfig = {
    variant: "simple",
    xKey: "value",
    yKey: "areacd",
    ySort: "ascending",
    dataLabels: {format: ",.0f"}
  }

  let stackedBarConfig = {
    variant: "stacked",
    height: 150,
    xKey: "value",
    yKey: "period",
    zKey: "areacd",
    yFormat: "%Y",
    yFormatDate: "%Y-%m-%d",
    ySort: "ascending",
    zSortKey: "E12000004"
  }

  let clusteredBarConfig = {
    variant: "clustered",
    xKey: "value",
    yKey: "areacd",
    zKey: "period",
    dataLabels: {format: ",.0f"},
    ySort: "descending",
    zSortKey: "2023-06-30"
  }
  
  let smBarConfig = {
    variant: "small-multiple",
    xKey: "value",
    yKey: "areacd",
    zKey: "period",
    xAxisTicks: 2,
    zFormat: "%Y",
    zFormatDate: "%Y-%m-%d",
    ySort: "descending"
  }

  function selectIndicator(selected) {
    indicator = !selected ? null : selected;
  }
</script>

<PhaseBanner phase="prototype"/>
<Header compact title="Svelte Plot demo" />
<Breadcrumb links={[{label: "ELS API experiments", href: `${base}/`}]}/>

<Section>
  <p style:margin="12px 0 32px">
    Select an indicator to view data as a map, bar and line chart. Chart data for each indicator will be lazy loaded when the chart comes into view on the page.
  </p>
  <form class="select-container" on:submit|preventDefault={() => selectIndicator(selected)}>
    <Select options={data.indicators} bind:value={selected} label="Select an indicator" placeholder="Eg. Household income"/>
    <Button small type="sumbit">Select area</Button>
  </form>
</Section>

{#if indicator}
  <Divider/>
  <NavSections>
    <NavSection title="Bar">
      <LazyLoad>
        <div class="chart-container">
          {#await fetchChartData(indicator.slug, "rgn")}
            Fetching chart data
          {:then chartData}
            <BarChart data={chartData} {...barConfig}/>
          {:catch}
            Failed to load chart data
          {/await}
        </div>
      </LazyLoad>
    </NavSection>
    <NavSection title="Stacked Bar">
      <LazyLoad>
        <div class="chart-container">
          {#await fetchChartData(indicator.slug, "rgn", "2021,2022,2023,2024")}
            Fetching chart data
          {:then chartData}
            <BarChart data={chartData} {...stackedBarConfig}/>
          {:catch}
            Failed to load chart data
          {/await}
        </div>
      </LazyLoad>
    </NavSection>
    <NavSection title="Clustered Bar">
      <LazyLoad>
        <div class="chart-container">
          {#await fetchChartData(indicator.slug, "rgn", "2023,2024")}
            Fetching chart data
          {:then chartData}
            {console.log(chartData)}
            <BarChart data={chartData} {...clusteredBarConfig}/>
          {:catch}
            Failed to load chart data
          {/await}
        </div>
      </LazyLoad>
    </NavSection>
        <NavSection title="Small Multiple Bar">
      <LazyLoad>
        <div class="chart-container">
          {#await fetchChartData(indicator.slug, "rgn", "2023,2024")}
            Fetching chart data
          {:then chartData}
            {console.log(chartData)}
            <BarChart data={chartData} {...smBarConfig}/>
          {:catch}
            Failed to load chart data
          {/await}
        </div>
      </LazyLoad>
    </NavSection>
    <!-- <NavSection title="Line">
      <LazyLoad>
        <div class="chart-container">
          {#await fetchChartData(indicator.slug, "ltla", "all")}
            Fetching chart data
          {:then chartData}
            <Line data={chartData}/>
          {:catch}
            Failed to load chart data
          {/await}
        </div>
      </LazyLoad>
    </NavSection> -->
  </NavSections>
{/if}

<Footer compact />

<style>
  :global(.ons-input) {
    color: #707070;
    margin-bottom: 10px;
  }
  .select-container {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    align-items: flex-end;
    width: 100%;
    gap: 6px;
  }
  .select-container > :global(div) {
    flex-grow: 1;
  }
  .select-container > :global(button) {
    flex-shrink: 1;
    padding-bottom: 4px;
  }
  .chart-container {
    display: block;
    width: 100%;
    min-height: 300px;
    margin-bottom: 32px;
  }
  .map-container {
    max-width: 400px;
  }
  .chart-container :global(svg) {
    overflow: visible;
  }
</style>