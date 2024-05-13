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
    // const response = await fetch('states-albers-10m.json');
    return us;
  }

  let svg;

  onMount(async () => {
    await loadData();

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
        svg
          .transition()
          .duration(750)
          .call(
            zoom.transform,
            d3.zoomIdentity,
            d3.zoomTransform(svg.node()).invert([width / 2, height / 2])
          );
      }

      function clicked(event, d) {
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

      document.getElementById("chart-container").appendChild(svg.node());
    });
  });
  // Mock function to demonstrate plot updating
  function updateLinePlot(data) {

    const svg = d3.select("#linePlot");
    svg.selectAll("*").remove(); // Clear previous plots

    // Here you should add the actual D3 code to create or update the line plot
    // This is a simplified placeholder
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

<!-- Add a placeholder for the line plot in your HTML -->
<svg id="linePlot" width="1000" height="400"></svg>

<div id="chart-container"></div>

<style>
  /* Add any necessary CSS styles here */
  .bar {
    fill: steelblue;
  }
</style>
