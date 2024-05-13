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
                .attr('transform', `translate(${width - 150}, 10)`) // Adjust the position
                .attr('id', 'close-button')
                .style('cursor', 'pointer')
                .on('click', reset); // Reset function on click

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
                .attr('transform', `translate(${width - 50}, 10)`) // Adjust the position
                .attr('id', 'overlay')
                .append('rect')
                .attr('width', 40)
                .attr('height', 40)
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
        background-color: rgba(255, 255, 255, 0.5); /* Semi-transparent white */
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
</div>
 -->

 <script>
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
</div>
