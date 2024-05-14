<script>
  import { onMount } from "svelte";
  import us from "./states-albers-10m.json";
  import * as d3 from "d3";
  import { feature, mesh } from "topojson-client";
  import { timeParse, timeFormat } from "d3";

  //   let data; // This will hold the CSV data
  let dataConfirm;
  let dataDeath;
  let currentData;
  let selectedStateData = []; // Data for the selected state
  const parseDate = d3.timeParse("%Y-%m");

  let columns; // To store date columns for x-axis labels
  //   let bool;
  // Load and parse CSV data
  async function loadData() {
    const response = await d3.csv("./state_monthly_confirm.csv");
    const response2 = await d3.csv("./state_monthly_death.csv");

    currentData = response;

    dataDeath = response2;

    columns = response.columns.slice(1); // Assuming first column is 'State'
    // console.log("Data loaded:", data);
    // console.log("Columns:", columns);
  }

  function updatePlot(stateName, data, bool) {
    const stateData = data.find((d) => d["2019-12"] === stateName);
    if (stateData) {
      const plotData = columns.map((date) => ({
        date, // Already the right format 'YYYY-MM'
        value: +stateData[date],
      }));
      updateLinePlot(plotData, stateName, bool);
    }
  }
  // Function to load and parse CSV
  //   async function loadData() {
  //     data = await d3.csv("./state_monthly_confirm.csv");
  //   }

  //   // Function to update plot data based on clicked state
  //   function updatePlot(stateName) {
  //     console.log("updata plot:", data); // Debug: Log data to see what's being passed.

  //     const stateData = data.find((d) => d.State === stateName);
  //     if (stateData) {
  //       const plotData = Object.entries(stateData)
  //         .slice(1)
  //         .map(([date, value]) => ({
  //           date,
  //           value: +value,
  //         }));
  //       updateLinePlot(plotData); // Update the plot with this data
  //     }
  //   }

  async function loadMapData() {
    // const response = await fetch('states-albers-10m.json');
    return us;
  }

  let svg;

  onMount(async () => {
    await loadData();
    // setupButtons();

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
        removeOverlay();
        removeGraph();
      }

      //   funcion switch_data(){

      //   }

      function clicked(event, d) {
        const overlay = document.getElementById("overlay");
        overlay.style.display = "block"; // Show the overlay
        overlay.style.filter = "blur(10px)"; // Apply blur effect
        const graph = document.getElementById("linePlot");
        graph.style.display = "block";
        const closeButton = document.getElementById("close-button");
        closeButton.addEventListener("click", () => {
          reset();
        });

        console.log("clicked", event, d);
        const [[x0, y0], [x1, y1]] = path.bounds(d);
        event.stopPropagation();
        states.transition().style("fill", null);
        d3.select(this).transition().style("fill", "red");

        // // confirm and death button
        // const confirmButton = document.getElementById();
        // confirmButton.addEventListener("click", () => {
        //   switch_data();
        // });

        // const deathButton = document.getElementById();
        // confirmButton.addEventListener("click", () => {
        //   switch_data();
        // });

        const stateName = d.properties.name;
        console.log("State clicked:", stateName);
        updatePlot(stateName, currentData, true);

        document
          .getElementById("confirmButton")
          .addEventListener("click", () => {
            updatePlot(stateName, currentData, true);
          });

        document.getElementById("deathButton").addEventListener("click", () => {
          updatePlot(stateName, dataDeath, false);
        });

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
        const overlay = document.getElementById("overlay");
        overlay.style.display = "none";
        overlay.style.filter = "none";
      }
      function removeGraph() {
        const graph = document.getElementById("linePlot");
        graph.style.display = "none";
      }

      const closeButton = svg
        .append("g")
        .attr("id", "close-button")
        .style("cursor", "pointer")
        .on("click", reset) // Reset function on click
        .attr("transform", `translate(${width - 150}, 10)`);

      closeButton
        .append("rect")
        .attr("width", 80)
        .attr("height", 40)
        .attr("rx", 10)
        .attr("ry", 10)
        .attr("fill", "#d6d6d6");

      closeButton
        .append("text")
        .attr("x", 40)
        .attr("y", 20)
        .attr("text-anchor", "middle")
        .attr("alignment-baseline", "middle")
        .text("Reset")
        .style("font-size", "24px")
        .style("fill", "#333");

      svg
        .append("g")
        .attr("id", "overlay")
        .append("rect")
        .attr("width", width)
        .attr("height", height)
        .attr("fill", "rgba(0, 0, 0, 0.5)") // Semi-transparent black
        .style("display", "none")
        .style("cursor", "pointer");


      document.getElementById("chart-container").appendChild(svg.node());
    });
  });
  // Mock function to demonstrate plot updating
  function updateLinePlot(data, stateName, isConfirmed) {
    const svg = d3.select("#linePlot");
    svg.selectAll("*").remove(); // Clear previous plots

    // Here you should add the actual D3 code to create or update the line plot
    // This is a simplified placeholder
    const margin = { top: 40, right: 20, bottom: 100, left: 100 },
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
        d3.axisBottom(x).tickValues(x.domain().filter((d, i) => i % 6 === 0))
      )
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end");

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

    if (isConfirmed) {
      svg
        .append("text")
        .attr("x", width / 2 + margin.left)
        .attr("y", margin.top / 2)
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .style("text-decoration", "underline")
        .text("COVID-19 Cases confirmed in " + stateName);
    } else {
      svg
        .append("text")
        .attr("x", width / 2 + margin.left)
        .attr("y", margin.top / 2)
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .style("text-decoration", "underline")
        .text("COVID-19 Deaths in " + stateName);
    }
  }
</script>

<!-- Add a placeholder for the line plot in your HTML -->
<!-- <svg id="linePlot" width="1000" height="400"></svg> -->
<!-- <button id="confirmButton">Confirm</button>
<button id="deathButton">Death</button> -->

<div id="chart-container">
    <!-- Place the overlay directly inside the container -->
    <div id="overlay"></div>

    <!-- Place the bar chart SVG after the overlay -->
    <svg id="linePlot" width="1000" height="400"></svg>
<button id="confirmButton">Confirm</button>
<button id="deathButton">Death</button>
</div>

<style>
  /* Add any necessary CSS styles here */
  .bar {
    fill: steelblue;
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
