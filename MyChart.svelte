<!-- <script>
    import { onMount } from 'svelte';
    import us from './states-albers-10m.json';
    import * as d3 from 'd3';
    import { feature, mesh } from 'topojson-client';
    
    async function loadMapData() {
        return us;
    }

    let svg;

    onMount(() => {
        loadMapData().then(us => {
            const width = 975;
            const height = 610;

            const zoom = d3.zoom()
                .scaleExtent([1, 8])
                .on("zoom", zoomed);

            svg = d3.create("svg")
                .attr("viewBox", [0, 0, width, height])
                .attr("width", width)
                .attr("height", height)
                .attr("style", "max-width: 100%; height: auto;")
                .on("click", reset);

            const path = d3.geoPath();

            const g = svg.append("g");

            const states = g.append("g")
                .attr("fill", "#444")
                .attr("cursor", "pointer")
                .selectAll("path")
                .data(feature(us, us.objects.states).features)
                .join("path")
                .on("click", clicked)
                .attr("d", path);

            states.append("title")
                .text(d => d.properties.name);

            g.append("path")
                .attr("fill", "none")
                .attr("stroke", "white")
                .attr("stroke-linejoin", "round")
                .attr("d", path(mesh(us, us.objects.states, (a, b) => a !== b)));

            svg.call(zoom);

            function reset() {
                states.transition().style("fill", null);
                svg.transition().duration(750).call(
                    zoom.transform,
                    d3.zoomIdentity,
                    d3.zoomTransform(svg.node()).invert([width / 2, height / 2])
                );
                removeOverlay(); // Remove overlay when resetting
            }

            function clicked(event, d) {
                const overlay = document.getElementById('overlay');
                overlay.style.display = 'block'; // Show the overlay
                overlay.style.filter = 'blur(10px)'; // Apply blur effect
                const closeButton = document.getElementById('close-button');
                closeButton.addEventListener('click', () => {
                    reset();
                });
                console.log("clicked", event, d);
                const [[x0, y0], [x1, y1]] = path.bounds(d);
                event.stopPropagation();
                states.transition().style("fill", null);
                d3.select(this).transition().style("fill", "red");
                svg.transition().duration(750).call(
                    zoom.transform,
                    d3.zoomIdentity
                        .translate(width / 2, height / 2)
                        .scale(Math.min(8, 0.9 / Math.max((x1 - x0) / width, (y1 - y0) / height)))
                        .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
                    d3.pointer(event, svg.node())
                );
            }

            function zoomed(event) {
                const { transform } = event;
                g.attr("transform", transform);
                g.attr("stroke-width", 1 / transform.k);
            }

            function removeOverlay() {
                const overlay = document.getElementById('overlay');
                overlay.style.display = 'none'; // Hide the overlay
                overlay.style.filter = 'none'; // Remove the blur effect
            }

            const closeButton = svg.append('g')
                .attr('id', 'close-button')
                .style('cursor', 'pointer')
                .on('click', reset) // Reset function on click
                .attr('transform', `translate(${width - 150}, 10)`); // Adjust the position

            closeButton.append('rect')
                .attr('width', 80)
                .attr('height', 40)
                .attr('rx', 10) // Adjust the horizontal radius
                .attr('ry', 10) // Adjust the vertical radius
                .attr('fill', '#d6d6d6');

            closeButton.append('text')
                .attr('x', 40) // Adjust the x position
                .attr('y', 20) // Adjust the y position
                .attr('text-anchor', 'middle')
                .attr('alignment-baseline', 'middle') // Align text to the middle
                .text('Reset')
                .style('font-size', '24px')
                .style('fill', '#333');

            svg.append('g')
                .attr('id', 'overlay')
                .append('rect')
                .attr('width', width)
                .attr('height', height)
                .attr('fill', 'rgba(0, 0, 0, 0.5)') // Semi-transparent black
                .style('display', 'none')
                .style('cursor', 'pointer');

            document.getElementById('chart-container').appendChild(svg.node());
        });
    });
</script>

<style>
    /* Add any necessary CSS styles here */

    #overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(255, 255, 255, 0.7); /* Semi-transparent white */
        display: none; /* Initially hidden */
        z-index: 200; /* Ensure it's above the SVG */
        pointer-events: none; /* Allow mouse events to pass through */
    }

    /* #close-button rect,
    #close-button text {
        pointer-events: auto;
        z-index: 10000;
    } */
    
</style>

<div id="chart-container">
    <div id="overlay"></div>
</div> -->


<script>
    import { onMount } from "svelte";
    import us from "./states-albers-10m.json";
    import * as d3 from "d3";
    import { feature, mesh } from "topojson-client";
    import { timeParse, timeFormat } from 'd3';
  
    let data; // This will hold the CSV data
    let selectedStateData = []; // Data for the selected state
    const parseDate = d3.timeParse("%Y-%m");
  
    // Function to load and parse CSV
    async function loadData() {
      data = await d3.csv("./state_monthly_confirm.csv");
    }
  
    // Function to update plot data based on clicked state
    function updatePlot(stateName) {
      console.log("updata plot:", data); // Debug: Log data to see what's being passed.
  
      const stateData = data.find((d) => d.State === stateName);
      if (stateData) {
        const plotData = Object.entries(stateData)
          .slice(1)
          .map(([date, value]) => ({
            date,
            value: +value,
          }));
        updateLinePlot(plotData); // Update the plot with this data
      }
    }
  
  
    async function loadMapData() {
      return us;
    }
  
    let svg;
  
    onMount(async () => {
        await loadData();
//updated  
        loadMapData().then((us) => {
            const width = 975;
            const height = 610;
    
            const zoom = d3.zoom().scaleExtent([1, 8]).on("zoom", zoomed);
    
            svg = d3
            .create("svg")
            .attr("viewBox", [0, 0, width, height])
            .attr("width", width)
            .attr("height", height)
            .attr("style", "max-width: 100%; height: auto;")
            .on("click", reset);
    
            const path = d3.geoPath();
    
            const g = svg.append("g");
    
            const states = g
            .append("g")
            .attr("fill", "#444")
            .attr("cursor", "pointer")
            .selectAll("path")
            .data(feature(us, us.objects.states).features)
            .join("path")
            .on("click", clicked)
            .attr("d", path);
    
            states.append("title").text((d) => d.properties.name);
    
            g.append("path")
            .attr("fill", "none")
            .attr("stroke", "white")
            .attr("stroke-linejoin", "round")
            .attr("d", path(mesh(us, us.objects.states, (a, b) => a !== b)));
    
            svg.call(zoom);
    
            function reset() {
                states.transition().style("fill", null);
                svg.transition().duration(750).call(
                    zoom.transform,
                    d3.zoomIdentity,
                    d3.zoomTransform(svg.node()).invert([width / 2, height / 2])
                );
                removeOverlay();
                removeGraph();
            }


//updated
            function clicked(event, d) {
                const overlay = document.getElementById('overlay');
                overlay.style.display = 'block'; // Show the overlay
                overlay.style.filter = 'blur(10px)'; // Apply blur effect
                const graph = document.getElementById('linePlot');
                graph.style.display = 'block';
                const closeButton = document.getElementById('close-button');
                closeButton.addEventListener('click', () => {
                    reset();
                });
                console.log("clicked", event, d);
                const [[x0, y0], [x1, y1]] = path.bounds(d);
                event.stopPropagation();
                states.transition().style("fill", null);
                d3.select(this).transition().style("fill", "red");
        
                const stateName = d.properties.name;
                console.log("State clicked:", stateName);
                updatePlot(stateName);
                svg
                    .transition()
                    .duration(750)
                    .call(
                    zoom.transform,
                    d3.zoomIdentity
                        .translate(width / 2, height / 2)
                        .scale(
                        Math.min(
                            8,
                            0.9 / Math.max((x1 - x0) / width, (y1 - y0) / height)
                        )
                        )
                        .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
                    d3.pointer(event, svg.node())
                    );
            }
    
            function zoomed(event) {
                const { transform } = event;
                g.attr("transform", transform);
                g.attr("stroke-width", 1 / transform.k);
            }
            function removeOverlay() {
                const overlay = document.getElementById('overlay');
                overlay.style.display = 'none';
                overlay.style.filter = 'none';
            }
            function removeGraph() {
                const graph = document.getElementById('linePlot');
                graph.style.display = 'none';
            }    
    
            const closeButton = svg.append('g')
                    .attr('id', 'close-button')
                    .style('cursor', 'pointer')
                    .on('click', reset) // Reset function on click
                    .attr('transform', `translate(${width - 150}, 10)`);

            closeButton.append('rect')
                .attr('width', 80)
                .attr('height', 40)
                .attr('rx', 10) 
                .attr('ry', 10)
                .attr('fill', '#d6d6d6');

            closeButton.append('text')
                .attr('x', 40)
                .attr('y', 20)
                .attr('text-anchor', 'middle')
                .attr('alignment-baseline', 'middle') 
                .text('Reset')
                .style('font-size', '24px')
                .style('fill', '#333');

            svg.append('g')
                .attr('id', 'overlay')
                .append('rect')
                .attr('width', width)
                .attr('height', height)
                .attr('fill', 'rgba(0, 0, 0, 0.5)') // Semi-transparent black
                .style('display', 'none')
                .style('cursor', 'pointer');
            document.getElementById("chart-container").appendChild(svg.node());
        });
    });

    function updateLinePlot(data) {
        const svg = d3.select("#linePlot");
        svg.selectAll("*").remove();
    
        const margin = { top: 20, right: 20, bottom: 30, left: 100 },
            width = 600 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;
    
        const x = d3
            .scaleBand()
            .range([0, width])
            .domain(data.map((d) => d.date))
            .padding(0.1);
    
        const y = d3
            .scaleLinear()
            .range([height, 0])
            .domain([0, d3.max(data, (d) => d.value)]);
    
    
        const g = svg
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);
    
        g.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(
            d3
                .axisBottom(x)
                .tickFormat(d3.timeFormat("%Y-%M"))
                .tickValues(x.domain().filter((d, i) => i % 12 === 0))
            );
    
        g.append("g").call(d3.axisLeft(y));
    
        g.selectAll(".bar")
            .data(data)
            .enter()
            .append("rect")
            .attr("class", "bar")
            .attr("x", (d) => x(d.date))
            .attr("width", x.bandwidth())
            .attr("y", (d) => y(d.value))
            .attr("height", (d) => height - y(d.value))
            .attr("fill", "steelblue");
    }
</script>

<!-- updated -->

<div id="chart-container">
    <!-- Place the overlay directly inside the container -->
    <div id="overlay"></div>

    <!-- Place the bar chart SVG after the overlay -->
    <svg id="linePlot" width="1000" height="400"></svg>
</div>



<style>
/* updated */
/* Add any necessary CSS styles here */
.bar {
    fill: steelblue;
}
#chart-container {
    position: relative; /* Position relative to contain the overlay */
    z-index: 100;
}

#overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(255, 255, 255, 0.7); /* Semi-transparent white */
    display: none; /* Initially hidden */
    z-index: 200; /* Ensure it's above the SVG */
    pointer-events: none; /* Allow mouse events to pass through */
}

#linePlot {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 300; /* Ensure it's above the overlay */
    pointer-events: none;
    display: none;
}

</style>
