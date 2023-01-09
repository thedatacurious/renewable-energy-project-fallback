<script>
  import { LayerCake, Svg, calcExtents, flatten } from "layercake";
  import Sankey from "./lib/Sankey.svelte";
  import Range from "./lib/Range.svelte";
  import SmallMultiple from "./lib/SmallMultiple.svelte";
  import { sankeyData, ChinaAfricaInfo, regionalNested } from "./data.js";

  import * as d3 from "d3";
  import { onMount } from 'svelte'; 
  import  {joinedData,continentsArray,beeswarm_data}  from './data.js';
  import Force from "./lib/Force.svelte";
  //import Beeswarm from "./Components/Charts/Beeswarm.svelte";
  

  $: sankeyYear = 2003;
  $: filteredSankeyData = sankeyData[sankeyYear];
  $: loanAmount = ChinaAfricaInfo[sankeyYear] ?? 0;

  const extentGetters = {
    x: (d) => d.year,
    y1: (d) => d.regionalInvestment,
    y2: (d) => d.regionalAvg,
  };

  const fullExtents = calcExtents(flatten(regionalNested), extentGetters);
  let chart = "Investment";

//Force and beeswarm
  let width=650;
  let height=200;
  let margin= {left: 40, right: 20, top: 20, bottom: 10};

  let year = 2013;
  let param_force='renewablesConsPerCap';

  let params_force=Object.keys(joinedData[0]).filter(d=>d.includes('renewables'))
  //renewablesConsPerCap
  let params_labels={
    renewablesCons: 'Primary energy consumption from renewables',
    renewablesConsChangePercPct: "Annual percentage change in energy consumption from renewables",
    renewablesConsChangePercTwh: "Annual change in other renewable consumption, measured in terawatt-hours",
    renewablesConsPerCap: "Per capita electricity consumption from renewables",
    renewablesGenPerCap: "Per capita electricity generation from renewables",
    renewablesShare: "Share of primary energy consumption that comes from renewables",
    renewables_share_elec: "Share of electricity generation that comes from renewables"
  }

  $ : beeswarm_data_filtered=beeswarm_data.filter((d)=>d.year==sankeyYear && d[param_force]);

  $ : filtered=joinedData.filter((d)=>d.year==sankeyYear && d[param_force]);

  let this_legend;
  let force_svg;

  onMount(() => 
  {
    let selectedCont=[];
   
   
    let colorScale = d3.scaleOrdinal()
                  .domain(continentsArray)
                  .range(d3.schemeCategory10);
 
   let legend=d3.select('.legend')
    .attr('transform', 'translate(' + (margin.left + margin.right + 60) + ',' + (margin.top - 20) + ')')
    .selectAll('g')
    .data(continentsArray)
    .enter()
    .append('g')
    .on('mouseover',function(e,g_data)
    {
      //the g
      console.warn(d3.select(this))
      d3.selectAll('.legend g').filter((d)=>
      {
        return d!==g_data;
      }).attr('opacity',.6)
    // console.info(d3.select(force_svg).selectAll('circle'))
      let circles=d3.select(force_svg).selectAll('circle');
      circles.attr('fill-opacity',.6)
      circles.each(function(d, i)
        {

        let data=JSON.parse(d3.select(this).attr('data'))
        if (data.continent!==g_data)
        d3.select(this).attr('fill-opacity',.2)
        })
    })
    .on('mouseout',function()
    {
      d3.selectAll('.legend g').attr('opacity',1)
      d3.select(force_svg).selectAll('circle').attr('fill-opacity',.6)
    })
  /*
  .on('click',function(e,g_data)
  {
    console.log(selectedCont)
    let pos=selectedCont.indexOf(g_data);
    if (pos==-1)
    {      
      //items 
      selectedCont.push(g_data)
    }
    else
    {      
       selectedCont.splice(pos, 1);
    }

    d3.selectAll('.legend g').each(function(d)
    {
      console.log(d)
      if (selectedCont.indexOf(d)>-1)
      d3.select(this).attr('opacity',.6)
      else
      d3.select(this).attr('opacity',1)
    })


  })
  */

  legend.append('circle')
  .attr('fill', (d, i) => 
  {
    return colorScale(d)
    })   
  .attr('transform','translate(0,8)')  
  .attr('r', 8)
  .attr('fill-opacity',.6)
  
  
  legend.append('text')
    .attr('x', 18)
    .attr('y', 10)
    .attr('dy', '.15em')
    .text((d, i) => d)
    .style('text-anchor', 'start')
    .style('font-size', 12);

    let sum=0;
    legend.each(function(d,i)
    {
      
      let node_width=(d3.select(this).node().getBBox().width);
      
      if (i>0)
      {
      
        d3.select(this).attr('transform','translate('+sum+', 5)')
        sum+=node_width+15;
      }
      
      else
      {
        
        sum+=node_width+15;
    d3.select(this).attr('transform','translate(8, 5)')
      }
    })
})
</script>

<main>
  <h1>The darker side of energy transition riches</h1>

  <p>
    Renewables are becoming an important source of energy over the years. Not
    only is it seen as a solution to our climate crisis—using renewables as
    opposed to other forms of gas such as coal/fossil fuels/oil releases less
    greenhouse gas—but reports also show that consumption of renewables can
    reduce income inequality.
  </p>

  <p>
    Countries where renewable energy use is close to half (47%) of total energy
    consumption will see income inequality decrease by 0.2% for every additional
    1% percent increase in renewable consumption, according to a study conducted
    by the <a href="https://www.sussex.ac.uk/broadcast/read/55693"
      >University of Sussex</a
    >.
  </p>

  <p>
    But data shows that progress has been unequal across continents, especially
    in developing countries and regions like Africa although they are rich in
    wind, solar, hydro, and geothermal energy sources.
  </p>

  <p>[Beeswarm plots placeholder]</p>

  <div class="select">
    <div>
      <select bind:value={param_force}>
        {#each Object.entries(params_labels) as p}
    
            <option id={p[0]} value={p[0]}>{p[1]}</option> 
        {/each}
      </select>
    </div>
<!--
    <div>
      <select bind:value={year}>
        {#each years as d }
            <option id={d}>{d}</option> 
        {/each}
      </select>
    </div>
    -->
  </div>
  <label for="basic-range">Years {sankeyYear} <small>From 2000 on</small></label>
<!--
<Range
 bind:my_data={filtered} 
  on:change={(e) => (year=e.detail.value)}
  id="basic-slider"
  min={2010}
  max={2021}
  initialValue={year}
/>
-->


    <svg class="legend" {width} height=50 xmlns:svg='https://www.w3.org/2000/svg' viewBox='0 0 {width} 50' bind:this={this_legend} />

    <svg class="force" {width} {height} xmlns:svg='https://www.w3.org/2000/svg' bind:this={force_svg} viewBox='0 0 {width} {height}'>
      <Force {params_labels} {width} {param_force} {continentsArray} {height} year={sankeyYear} bind:my_data={filtered}/> 
    </svg>
    
    
  <p>
    We take a look at how much investment each region is receiving for renewable
    energy from 2000 to 2020. Despite the high investment amount in Sub-Saharan
    Africa, the share of primary energy from renewable sources has been pretty
    stagnant.
  </p>

  <div class="input-container">
    <label
      ><input type="radio" bind:group={chart} value="Investment" />Investment
      received</label
    >
    <label
      ><input type="radio" bind:group={chart} value="Usage" />Usage of renewable
      energy</label
    >
  </div>

  <div class="small-multiple-container">
    {#each regionalNested as data}
      <div class="line-container">
        <SmallMultiple {data} {fullExtents} {chart} {extentGetters} />
      </div>
    {/each}
  </div>

  <p>
    Upon of closer analysis of renewable energy finance flows from donors (left)
    to regional recipients (right), we see that China has been one of the
    biggest donors to Africa from 2003.
  </p>

  {#if loanAmount != 0}
    <p>
      However, the flows are in the form of loans, as opposed to other finance
      types like grants. In {sankeyYear}, China issued US${loanAmount} mil of loans
      to African countries.
    </p>
  {:else}
    <p>
      However, the flows are in the form of loans, as opposed to other finance
      types like grants. There was no record of finance flows from China to
      African countries in {sankeyYear} in
      <a
        href="https://www.irena.org/Data/View-data-by-topic/Finance-and-Investment/Renewable-Energy-Finance-Flows"
        >IRENA database</a
      >.
    </p>
  {/if}

  <p>
    <a
      href="https://www.sciencedirect.com/science/article/pii/S0305750X20304939"
      >Some studies</a
    > of Chinese investment renewable projects caution against overly optimistic
    expectations of co-benefits.
  </p>

  <div>
    <label for="basic-range">Year</label>
    <Range
      on:change={(e) => (sankeyYear = e.detail.value)}
      id="basic-slider"
      min={2000}
      max={2020}
      initialValue={2003}
    />
  </div>
  {sankeyYear}

  <div class="sankey-container">
    <LayerCake data={filteredSankeyData}>
      <Svg>
        <Sankey />
      </Svg>
    </LayerCake>
  </div>
</main>

<style>
  .small-multiple-container {
    height: 600px;
    width: 100%;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
  .input-container {
    margin-top: 7px;
  }
  label {
    cursor: pointer;
  }
  input {
    margin-right: 7px;
  }
  .line-container {
    width: 15%;
    height: 40%;
  }

  .sankey-container {
    width: 100%;
    height: 800px;
  }

  
  

.legend
{
  cursor:pointer;
}
 .select select 
 {
  padding: 5px;
    font-size: unset;
    font-family: unset;
    max-width: 55%;
    float: left;
        margin-right: 10px;
            margin-top: 10px;
 }

  :global(.tooltip)
  {
    font-size:.8rem;
  }

  :global(.tooltip span)
  {
    display:block;
    
  }
  main {
    padding: 1em;
    padding-top: 5rem;
    margin: 0 auto;
  }

  :global(circle:hover)
 {
  cursor:pointer;
 }
  :global(.beeswarm .axis text,.yAxis text)
  {
    fill:white;
    font-size:.7rem;
  }
  :global(.highlighted)
  
    {
      padding:2px;
      background:#ff9800;
      color:white;
    }
  :global(.force),:global(.beeswarm)
   {
    
    margin-bottom:3rem;
  }

</style>
