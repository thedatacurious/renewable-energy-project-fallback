<script>
    import * as d3 from "d3";
    import {interpolateInferno} from "https://cdn.skypack.dev/d3-scale-chromatic@3";    
    export let width;
    export let height;
    
    import Axis from '../utils/Axis.svelte';
    
    export let year;
    export let param_force;
    export let params_labels;
    export let my_beeswarmdata;
    export let regionsArray;
    export let continentsArray;
    export let forceY_param;
    //='targetRegion';

    let arr_force;
    
    if (forceY_param=='targetRegion')
    arr_force=regionsArray;
      else
      arr_force=continentsArray;
  
    let selected_datapoint;
    let pinYAxis; // declare pins
    let pinXAxis;
    let verticalLine;
    
    let padding = 0.5;
    let margin= {left: 120, right: 20, top: 0, bottom: 50};
    //height - margin.bottom - margin.top
    let nodes;
    let all_nodes;
    
    //$ : values=my_beeswarmdata; 
  
    $ : xScale =d3.scaleLinear()
      .domain([d3.min(my_beeswarmdata,d=>d[param_force]),d3.max(my_beeswarmdata,d=>d[param_force])])
      .nice()
      .range([margin.left, (width - margin.right-margin.left)])
  
    
    $ : radiusScale = d3.scaleSqrt()
      .domain([d3.min(my_beeswarmdata,d=>d.gdpPerCap),d3.max(my_beeswarmdata,d=>d.gdpPerCap)])
      //.nice()
      .range([2, 8])  
    
    $ : yScale = d3.scaleBand()
      .domain(arr_force)
      .range([(height + margin.bottom), margin.bottom])    
     // .range([height + margin.bottom -50, margin.top-50])
              
     // call axis generators on the scale and pin the SVG pins.
      $: if (pinYAxis) {
        
        d3.select(pinYAxis).call(d3.axisLeft(yScale))
     
      }
      $: if (pinXAxis) {
        
        d3.select(pinXAxis).call(d3.axisBottom(xScale))
 
      }
    
      //we need more than 10 or 20 colors
      $ : colorScale = d3.scaleSequential().domain([0,arr_force.length])
        .interpolator(interpolateInferno);


  function find_arr (region)
    {
      
     // console.log(region,regionsArray.indexOf(region),colorScale(regionsArray.indexOf(region)))
      return colorScale(arr_force.indexOf(region))
    }
    
    let tooltip = d3.select('body')
          .append('div')          
          .attr('class', 'beeswarm-tooltip tooltip')
          .style('position', 'absolute')
          .style('z-index', '1')
          .style('visibility', 'hidden')
          .style('padding', '10px')
          .style('background', 'rgba(0,0,0,0.6)')
          .style('border-radius', '4px')
          .style('color', '#fff');
    
    
    function mouseover_fx(e,d) 
        {
          let all=d3.select(all_nodes).selectAll('circle');
          console.log(all.selectAll('increm').size())
          
          all.attr('opacity',.5)
  
          let _this=d3.select(e.target);
          _this.attr('opacity',1);
  
          let this_r=d3.select(e.target).attr('r')
          _this
          .classed('increm',true)
          .transition()
          .duration(500)
          .attr('r',this_r*2)
          .on('end',()=>
          {
            _this
            .classed('increm',false)
            .transition()
          .duration(100)
          
          .attr('r',this_r)
          })       
          
          all.attr('opacity',1);
  /*
          var _line=d3.select(verticalLine);
          _line.attr("x1", _this.attr("cx"))
          .attr("y1", _this.attr("cy"))
          .attr("y2", (height - margin.bottom))
          .attr("x2",  _this.attr("cx"))
          .attr("opacity", 1);
          */
  
          
          const tooltipWidth = tooltip.node().offsetWidth;
          const tooltipHeight = tooltip.node().offsetHeight;
          tooltip
                                .style("left", e.pageX - tooltipWidth/2 +'px')
                                .style("top", e.pageY-tooltipHeight - 10+'px')
                                .style('visibility', 'visible')
                                
                                .html(`<b>Region</b>: ${d.targetRegion} <br/>
                                <b>Country</b>: ${d.country} <br/>
                                <b>Continent</b>: ${d.continent} <br/>
                                <b>Year</b>: ${d.year} <br/>
                                <b>GDP per cap</b>: ${d.gdp.toLocaleString('en-US', {maximumFractionDigits: 2})}<br/>
                                <b>% of primary energy renewables consumption</b>: ${d['renewablesShareCon']}<br/>
                                <b>`+params_labels[param_force]+`</b>: ${d[param_force]}
                                
                                `)
        }
       
    
       let BeedisplayData = [];        
   

       function update() {        
        BeedisplayData = my_beeswarmdata
        return my_beeswarmdata
      }
    
        console.log(my_beeswarmdata)
        console.info(param_force)
    
        $:simulation = d3.forceSimulation(my_beeswarmdata)
        .force('x', d3.forceX((d) => xScale(d[param_force])).strength(2))
        .force('y', d3.forceY((d) => yScale(d[forceY_param])).strength(.3))
        .force('collide', d3.forceCollide(d => radiusScale(d.gdpPerCap) + padding).strength(.3))    
        //.tick();
      
        $: simulation.on("tick", update)
      
    </script>
    
   
  
    <g
        class="yAxis"
        bind:this={pinYAxis}
        transform="translate({margin.left},-{margin.bottom})"
      />

      <g
        class="xAxis"
        bind:this={pinXAxis}
        
        transform="translate(0,{(height -margin.bottom/2)+8})"
      />
     <!-- <Axis {width} {height} {margin} scale={xScale} position='bottom' />  -->
    <!-- <Axis {width} {height} {margin} scale={yScale} position='left' />  -->
           
    <g class='circles' bind:this={all_nodes} transform="translate(0,-35)">
    
    {#each BeedisplayData as d,i}
    
    <circle bind:this={nodes}
        r={radiusScale(d.gdpPerCap) }
        stroke="black"
        stroke-width=.5
        fill={find_arr(d[forceY_param])}
        fill-opacity={1}
        cx={d.x}
        cy={d.y}
        on:mouseout={(e)=>
        {
         // d3.select(verticalLine).attr('opacity',0);
        }}
        on:mouseover={(e) => {selected_datapoint = d; mouseover_fx(e,d)}}
        on:mousemove={(e)=>{ 
  
          
          const tooltipWidth = tooltip.node().offsetWidth;
          const tooltipHeight = tooltip.node().offsetHeight;
          tooltip
            .style("left", e.pageX - tooltipWidth/2 +'px')
            .style("top", e.pageY-tooltipHeight - 10+'px')
        }}
        on:mouseout={(e)=>{ 
          tooltip.style('visibility', 'hidden')
        }
      }
    
    
    />
    
    {/each}
    </g>
  
    <!-- <line bind:this={verticalLine} stroke="grey"/> -->
  
 
    
    
    
        
    
