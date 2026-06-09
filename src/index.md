---
toc: false
---

# Titanic Survival Explorer

CSC 477 Assignment 5

**Question:** How did passenger age, fare, sex, and class relate to survival on the Titanic?

<div class="ocean">
  <div class="ship">🚢</div>
</div>

<style>
.ocean {
  width: 100%;
  height: 120px;
  position: relative;
  overflow: hidden;
  border-bottom: 3px solid #4da6ff;
  margin-bottom: 2rem;
}

.ship {
  position: absolute;
  font-size: 64px;
  animation: sail 12s linear infinite;
}

.controls-row {
  display: grid;
  grid-template-columns: repeat(5, minmax(120px, 1fr));
  gap: 1rem;
  align-items: end;
  margin-bottom: 1.5rem;
}

.chart-row {
  display: grid;
  grid-template-columns: minmax(0, 1.3fr) minmax(0, 0.9fr);
  gap: 1.5rem;
  align-items: start;
}

.chart-row svg {
  max-width: 100%;
  height: auto;
}

@media (max-width: 1000px) {
  .controls-row {
    grid-template-columns: repeat(2, minmax(120px, 1fr));
  }

  .chart-row {
    grid-template-columns: 1fr;
  }
}

@keyframes sail {
  from { left: -100px; }
  to { left: 100%; }
}
</style>

```js
const titanic = await FileAttachment("data/Titanic-Dataset.csv").csv({typed: true});
```

```js
const titanicClean = titanic.filter(
  d => d.Age != null && d.Fare != null
);
```

```js
const numericOptions = ["Age", "Fare", "Pclass", "SibSp", "Parch"];

const niceLabel = d =>
  d === "Age" ? "Age" :
  d === "Fare" ? "Fare paid (£)" :
  d === "Pclass" ? "Passenger class" :
  d === "SibSp" ? "Siblings/spouses aboard" :
  d === "Parch" ? "Parents/children aboard" :
  d;
```

```js
const pageWidth = width;
const scatterWidth = Math.max(520, Math.min(700, pageWidth * 0.58));
const barWidth = Math.max(360, Math.min(500, pageWidth * 0.36));
const chartHeight = 450;
```

## Chart Controls

<div class="controls-row">

```js
const selectedSex = view(
  Inputs.select(
    ["all", "male", "female"],
    {
      label: "Filter by sex",
      format: d => d === "all" ? "All passengers" : d
    }
  )
);
```

```js
const selectedClass = view(
  Inputs.select(
    ["all", 1, 2, 3],
    {
      label: "Filter by passenger class",
      value: "all",
      format: d => d === "all" ? "All classes" : `Class ${d}`
    }
  )
);
```

```js
const xVariable = view(
  Inputs.select(numericOptions, {
    label: "X-axis",
    value: "Age",
    format: niceLabel
  })
);
```

```js
const yVariable = view(
  Inputs.select(numericOptions, {
    label: "Y-axis",
    value: "Fare",
    format: niceLabel
  })
);
```

```js
const colorBy = view(
  Inputs.select(
    ["Survived", "Sex", "Pclass"],
    {
      label: "Color by",
      value: "Survived",
      format: d =>
        d === "Survived" ? "Survival status" :
        d === "Sex" ? "Sex" :
        "Passenger class"
    }
  )
);
```

</div>

```js
const filteredTitanic = titanicClean.filter(d => {
  const sexMatch =
    selectedSex === "all" || d.Sex === selectedSex;

  const classMatch =
    selectedClass === "all" || d.Pclass === selectedClass;

  return sexMatch && classMatch;
});
```

<div class="chart-row">

<div>

## Interactive Passenger Comparison

Each dot represents one Titanic passenger.

```js
{
  const width = scatterWidth;
  const height = chartHeight;
  const margin = {top: 50, right: 125, bottom: 65, left: 80};

  const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height);

  svg.append("text")
    .attr("x", margin.left)
    .attr("y", 28)
    .attr("font-size", 16)
    .attr("font-weight", "bold")
    .text(`Each dot is a Titanic passenger`);

  const xExtent = d3.extent(filteredTitanic, d => d[xVariable]);
  const yExtent = d3.extent(filteredTitanic, d => d[yVariable]);

  const x = d3.scaleLinear()
    .domain([0, xExtent[1]])
    .nice()
    .range([margin.left, width - margin.right]);

  const y = d3.scaleLinear()
    .domain([0, yExtent[1]])
    .nice()
    .range([height - margin.bottom, margin.top]);

  const color = colorBy === "Survived"
    ? d3.scaleOrdinal([0, 1], ["#7570b3", "#1b9e77"])
    : d3.scaleOrdinal(d3.schemeTableau10);

  const xAxis = svg.append("g")
    .attr("transform", `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x));

  const yAxis = svg.append("g")
    .attr("transform", `translate(${margin.left},0)`)
    .call(d3.axisLeft(y));

  svg.append("text")
    .attr("x", width / 2)
    .attr("y", height - 18)
    .attr("text-anchor", "middle")
    .text(niceLabel(xVariable));

  svg.append("text")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", 24)
    .attr("text-anchor", "middle")
    .text(niceLabel(yVariable));

  const points = svg.append("g");

  points.selectAll("circle")
    .data(filteredTitanic)
    .join("circle")
    .attr("cx", d => x(d[xVariable]))
    .attr("cy", d => y(d[yVariable]))
    .attr("r", 4.5)
    .attr("opacity", 0.7)
    .attr("fill", d => colorBy === "Survived" ? color(d.Survived) : color(d[colorBy]))
    .append("title")
    .text(d =>
      `${d.Name}
Sex: ${d.Sex}
Age: ${d.Age}
Fare: £${d.Fare}
Class: ${d.Pclass}
Survival: ${d.Survived === 1 ? "Survived" : "Did not survive"}`
    );

  const legendValues =
    colorBy === "Survived" ? [0, 1] :
    colorBy === "Sex" ? ["male", "female"] :
    [1, 2, 3];

  const legend = svg.append("g")
    .attr("transform", `translate(${width - margin.right + 25}, ${margin.top})`);

  legend.append("text")
    .attr("font-weight", "bold")
    .attr("y", -10)
    .text(colorBy === "Survived" ? "Survival" : colorBy === "Sex" ? "Sex" : "Class");

  legend.selectAll("circle")
    .data(legendValues)
    .join("circle")
    .attr("cx", 0)
    .attr("cy", (d, i) => i * 24 + 10)
    .attr("r", 6)
    .attr("fill", d => color(d));

  legend.selectAll("text.legend-label")
    .data(legendValues)
    .join("text")
    .attr("x", 14)
    .attr("y", (d, i) => i * 24 + 15)
    .text(d =>
      colorBy === "Survived" ? (d === 1 ? "Survived" : "Did not survive") :
      colorBy === "Pclass" ? `Class ${d}` :
      d
    );

  const zoom = d3.zoom()
    .scaleExtent([1, 8])
    .translateExtent([[margin.left, margin.top], [width - margin.right, height - margin.bottom]])
    .extent([[margin.left, margin.top], [width - margin.right, height - margin.bottom]])
    .on("zoom", event => {
      const zx = event.transform.rescaleX(x);
      const zy = event.transform.rescaleY(y);

      xAxis.call(d3.axisBottom(zx));
      yAxis.call(d3.axisLeft(zy));

      points.selectAll("circle")
        .attr("cx", d => zx(d[xVariable]))
        .attr("cy", d => zy(d[yVariable]));
    });

  svg.call(zoom);

  display(svg.node());
}
```

</div>

<div>

## Survival Rate by Passenger Class

This bar chart updates based on the sex filter while keeping all passenger classes visible.

```js
const classChartData = titanicClean.filter(d =>
  selectedSex === "all" || d.Sex === selectedSex
);

const classSummary = d3.rollups(
  classChartData,
  v => ({
    passengers: v.length,
    survivalRate: d3.mean(v, d => d.Survived) * 100
  }),
  d => d.Pclass
).map(([Pclass, values]) => ({
  Pclass: `Class ${Pclass}`,
  passengers: values.passengers,
  survivalRate: values.survivalRate
})).sort((a, b) => d3.ascending(a.Pclass, b.Pclass));
```

```js
{
  const width = barWidth;
  const height = chartHeight;
  const margin = {top: 50, right: 25, bottom: 65, left: 75};

  const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height);

  svg.append("text")
    .attr("x", margin.left)
    .attr("y", 28)
    .attr("font-size", 16)
    .attr("font-weight", "bold")
    .text("Survival rate by class");

  const x = d3.scaleBand()
    .domain(classSummary.map(d => d.Pclass))
    .range([margin.left, width - margin.right])
    .padding(0.25);

  const y = d3.scaleLinear()
    .domain([0, 100])
    .range([height - margin.bottom, margin.top]);

  svg.append("g")
    .attr("transform", `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x));

  svg.append("g")
    .attr("transform", `translate(${margin.left},0)`)
    .call(d3.axisLeft(y).tickFormat(d => `${d}%`));

  svg.append("text")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", 24)
    .attr("text-anchor", "middle")
    .text("Survival rate (%)");

  svg.selectAll("rect")
    .data(classSummary)
    .join("rect")
    .attr("x", d => x(d.Pclass))
    .attr("y", d => y(d.survivalRate))
    .attr("width", x.bandwidth())
    .attr("height", d => y(0) - y(d.survivalRate))
    .attr("fill", "#4e79a7")
    .append("title")
    .text(d =>
      `${d.Pclass}
Passengers: ${d.passengers}
Survival rate: ${d.survivalRate.toFixed(1)}%`
    );

  svg.selectAll("text.bar-label")
    .data(classSummary)
    .join("text")
    .attr("x", d => x(d.Pclass) + x.bandwidth() / 2)
    .attr("y", d => y(d.survivalRate) - 7)
    .attr("text-anchor", "middle")
    .attr("font-size", 12)
    .text(d => `${d.survivalRate.toFixed(1)}%`);

  display(svg.node());
}
```

</div>

</div>

```js
const passengerCount = filteredTitanic.length;
const survivedCount = filteredTitanic.filter(d => d.Survived === 1).length;
const diedCount = passengerCount - survivedCount;
const survivalRate = passengerCount > 0 ? (survivedCount / passengerCount) * 100 : 0;
const avgAge = d3.mean(filteredTitanic, d => d.Age);
const avgFare = d3.mean(filteredTitanic, d => d.Fare);
```

## Current Selection Dashboard

<div class="grid grid-cols-4">
  <div class="card">
    <h2>${passengerCount}</h2>
    <p>Passengers shown</p>
  </div>

  <div class="card">
    <h2>${survivedCount}</h2>
    <p>Survived</p>
  </div>

  <div class="card">
    <h2>${diedCount}</h2>
    <p>Did not survive</p>
  </div>

  <div class="card">
    <h2>${survivalRate.toFixed(1)}%</h2>
    <p>Survival rate</p>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>${avgAge ? avgAge.toFixed(1) : "N/A"}</h2>
    <p>Average age</p>
  </div>

  <div class="card">
    <h2>${avgFare ? `£${avgFare.toFixed(2)}` : "N/A"}</h2>
    <p>Average fare</p>
  </div>
</div>

```js
const analysisText =
  survivalRate > 50
    ? "More than half of the currently displayed passengers survived."
    : "Less than half of the currently displayed passengers survived.";
```

**Analysis:** ${analysisText}

Loaded ${titanic.length} passengers.

# Design Rationale

First of all, I thought that the topic itself would be kind of interesting. I like the movie Titanic (at least until it starts sinking), and I thought I could get some good interactions out of it. I wanted to start off with a little boat going across the screen just for something fun. I remember one of the visualizations we looked at had blocks moving across the screen depending on the forces acting on it, so I wanted to have something moving across the page to create a similar aesthetic and make the visualization feel more engaging.

I also wanted to create an adjustable scatterplot to make the visualization more interactive and interesting. Users can zoom in on the chart, switch the x-axis and y-axis variables, filter by sex and passenger class, and change how the passengers are colored. I wanted these controls to encourage exploration and help users investigate different relationships in the dataset. To make the visualization easier to understand, I used more descriptive labels for the variables and included units for fare values so users can immediately understand what they are looking at. Each point in the scatterplot represents a passenger, and the title and labels help communicate that clearly.

In order to help guide the user, I added a dashboard that gives summary statistics about the passengers currently being displayed. I think this helps users better understand what is happening in the visualization and gives them more confidence while exploring the data. The dashboard updates automatically as the filters change, allowing users to quickly see how different groups compare.

Finally, I wanted to include a second chart that focuses on survival rates by passenger class. Survival is the main message I am trying to highlight throughout the visualization, so I thought it was important to include a separate view that makes those comparisons easy to see. The two charts work together so that users can explore individual passengers in the scatterplot while also seeing broader trends in the survival rate chart. I also chose colors that clearly distinguish categories while remaining accessible to a wider range of users.

I considered using only a single chart, but I decided that pairing the scatterplot with a survival-rate chart would make it easier to see both individual passengers and overall trends. I also considered using separate charts for different passenger groups, but I felt that filters would give users more flexibility while keeping the interface simpler.

Overall, my goal was to create an interactive visualization that is both informative and enjoyable to explore, while helping users better understand how factors such as age, fare, sex, and passenger class related to survival on the Titanic.


# References

Dataset Source:

Titanic Dataset by M Yasser H

https://www.kaggle.com/datasets/yasserh/titanic-dataset

Visualization Inspiration:

CSC 477 course materials, lectures, and in-class examples.

AI Assistance:

OpenAI ChatGPT was used to assist with debugging Observable Framework code, resolving implementation errors, and troubleshooting visualization display issues. All design decisions, interaction choices, and written rationale were created fully by the author.

## Project Repository

GitHub Repository:

<a href="https://github.com/jillian-Bogosian/A5--titanic-viz" target="_blank">
View Source Code
</a>