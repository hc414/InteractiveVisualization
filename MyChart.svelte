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
        removeButtons();
      }

      //   funcion switch_data(){

      //   }

      function clicked(event, d) {
        showPlot();

        const overlay = document.getElementById("overlay");
        overlay.style.display = "block"; // Show the overlay
        overlay.style.filter = "blur(10px)"; // Apply blur effect
        const graph = document.getElementById("linePlot");
        graph.style.display = "block";
        const butt = document.getElementById("buttons-container");
        butt.style.display = "grid";
        const resetButton = document.getElementById("resetButton");
        resetButton.addEventListener("click", () => {
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

      function removeButtons() {
        const buttons = document.getElementById("buttons-container");
        buttons.style.display = "none";
      }

      function showPlot() {
        document.getElementById("overlay").style.display = "block";
        document.getElementById("linePlot").style.display = "block";
      }

      function hidePlot() {
        document.getElementById("overlay").style.display = "none";
        document.getElementById("lineRequest").style.display = "none";
      }
      //   const closeButton = svg
      //     .append("g")
      //     .attr("id", "close-button")
      //     .style("cursor", "pointer")
      //     .on("click", reset) // Reset function on click
      //     .attr("transform", `translate(${width - 150}, 10)`);

      //   closeButton
      //     .append("rect")
      //     .attr("width", 80)
      //     .attr("height", 40)
      //     .attr("rx", 10)
      //     .attr("ry", 10)
      //     .attr("fill", "#d6d6d6");

      //   closeButton
      //     .append("text")
      //     .attr("x", 40)
      //     .attr("y", 20)
      //     .attr("text-anchor", "middle")
      //     .attr("alignment-baseline", "middle")
      //     .text("Reset")
      //     .style("font-size", "24px")
      //     .style("fill", "#333");

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

    // g.selectAll(".bar")
    //   .data(data)
    //   .enter()
    //   .append("rect")
    //   .attr("class", "bar")
    //   .attr("x", (d) => x(d.date))
    //   .attr("width", x.bandwidth())
    //   .attr("y", (d) => y(d.value))
    //   .attr("height", (d) => height - y(d.value))
    //   .attr("fill", "steelblue");

    const xAxisLabel = svg
      .append("text")
      .attr("class", "axis-label")
      .attr("text-anchor", "middle") // Center the text
      .attr("x", width / 2 + 100) // Center horizontally
      .attr("y", height + margin.bottom + 10) // Position below the x-axis
      .text("Date"); // The label text
    
    const yAxisLabel = svg
      .append("text")
      .attr("class", "axis-label")
      .attr("text-anchor", "middle") // This will align the center of the text with the specified coordinates
      .attr(
        "transform",
        `translate(${-margin.left + 130}, ${height / 2 + 40}) rotate(-90)`
      ) // Moves the label to the left and rotates it
      .text("Number of Cases"); // The text for the label

    if (isConfirmed) {
      svg
        .append("text")
        .attr("x", width / 2 + margin.left)
        .attr("y", margin.top / 2)
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .style("text-decoration", "underline")
        .text("Cumulative COVID-19 Case Count in " + stateName);
      g.selectAll(".line")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "line")
        .attr("x", (d) => x(d.date))
        .attr("width", x.bandwidth())
        .attr("y", (d) => y(d.value))
        .attr("height", (d) => height - y(d.value))
        .attr("fill", "steelblue");
    } else {
      svg
        .append("text")
        .attr("x", width / 2 + margin.left)
        .attr("y", margin.top / 2)
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .style("text-decoration", "underline")
        .text("Cumulative COVID-19 Death Count in  " + stateName);
      g.selectAll(".bar")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "bar")
        .attr("x", (d) => x(d.date))
        .attr("width", x.bandwidth())
        .attr("y", (d) => y(d.value))
        .attr("height", (d) => height - y(d.value))
        .attr("fill", "red");
    }
  }
</script>

<!-- Add a placeholder for the line plot in your HTML -->
<!-- <svg id="linePlot" width="1000" height="400"></svg> -->
<!-- <button id="confirmButton">Confirm</button>
<button id="deathButton">Death</button> -->

<!-- <div id="chart-container">
  <div id="overlay"></div>

  <div id="plot-and-controls">
    <svg id="linePlot" width="800" height="400"></svg>
    <div id="reset">
      <button id="resetButton">Reset</button>
    </div>

    <div id="buttons-container">
      <button id="confirmButton">Confirm</button>
      <button id="deathButton">Death</button>
    </div>
  </div>
</div> -->
<div id="chart-container">
  <div id="overlay"></div>
  <div id="plot-and-controls" style="position: absolute;">
    <svg id="linePlot" width="750" height="410"></svg>
    <div id="reset">
      <button id="resetButton">Reset</button>
    </div>
    <div id="buttons-container">
      <button id="confirmButton">Confirm</button>
      <button id="deathButton">Death</button>
    </div>
  </div>
</div>

<style>
  /* Add any necessary CSS styles here */

  #chart-container {
    /* display: absolute;
    justify-content: center;
    align-items: center;
    position: relative; 
    width: 100%;
    height: 100vh; */
    width: 100%;
    max-width: 100%;
    height: auto;
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden;
  }

  #overlay {
    position: fixed; /* Use fixed instead of absolute to ensure it covers the whole viewport */
    top: 0;
    left: 0;
    width: 100vw; /* Full viewport width */
    height: 100vh; /* Full viewport height */
    background-color: rgba(55, 54, 54, 0.7); /* Semi-transparent black */
    display: none; /* Initially hidden */
    z-index: 200; /* High z-index to ensure it is above other content */
    pointer-events: auto; /* Typically, you might want this disabled to allow clicks to pass through when hidden */
  }
  /* #overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 975px;
    height: 610px;
    background-color: rgba(55, 54, 54, 0.7); 
    display: none; 
    z-index: 200;
    pointer-events: auto; 
  } */

  /* #linePlot {
    position: absolute;
    top: 10%;
    left: 10%;
    width: 750px;
    height: 410px;
    z-index: 300;
    pointer-events: none;
    display: none;
    margin: auto;
    transform: translate(-50, 50);
    background-color: white;
  } */

  #linePlot {
    position: absolute;
    top: 50%; /* Position the top edge of the element at the center of the viewport */
    left: 50%; /* Position the left edge of the element at the center of the viewport */
    width: 750px;
    height: 410px;
    z-index: 300;
    pointer-events: none;
    display: none;
    margin: auto;
    transform: translate(
      -50%,
      -50%
    ); /* Adjust the position of the element to center it exactly */
    background-color: white;
  }

  #plot-and-controls {
    display: flex; /* Aligns children (SVG and buttons container) horizontally */
    z-index: 300; /* Ensures it's above the overlay */
  }

  #buttons-container {
    position: absolute; /* Position it relative to the nearest positioned ancestor */
    bottom: 80px; /* Position at the top with some margin */
    left: 250px; /* Align it to the right side */
    z-index: 400; /* Higher z-index to be on top of other elements */
    display: none;
    flex-direction: column; /* Buttons stacked vertically */
    overflow: auto;
    background-color: rgba(
      255,
      255,
      255,
      0.9
    ); /* Optional: add a slight background */
  }

  #reset {
    position: absolute; /* Position it relative to the nearest positioned ancestor */
    bottom: 263px; /* Position at the top with some margin */
    left: 430px; /* Align it to the right side */
    z-index: 400; /* Higher z-index to be on top of other elements */
    display: flex;
  }
  /* button {
    margin-bottom: 8px;
    cursor: pointer; 
  } */

  /* #buttons-container {
  display: flex;
  flex-direction: column; 
  /* padding-left: 20px; 
}  */
</style>
