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
const selectedSex = view(
  Inputs.select(
    ["all", "male", "female"],
    {label: "Filter by sex"}
  )
);
```

```js
const selectedClass = view(
  Inputs.select(
    ["all", 1, 2, 3],
    {
      label: "Filter by Passenger Class",
      value: "all"
    }
  )
);
```

```js
const numericOptions = ["Age", "Fare", "Pclass", "SibSp", "Parch"];
```

```js
const xVariable = view(
  Inputs.select(numericOptions, {
    label: "X-axis",
    value: "Age"
  })
);
```

```js
const yVariable = view(
  Inputs.select(numericOptions, {
    label: "Y-axis",
    value: "Fare"
  })
);
```

```js
const colorBy = view(
  Inputs.select(
    ["Survived", "Sex", "Pclass"],
    {
      label: "Color By",
      value: "Survived"
    }
  )
);
```

```js
const filteredTitanic = titanicClean.filter(d => {
  const sexMatch =
    selectedSex === "all" || d.Sex === selectedSex;

  const classMatch =
    selectedClass === "all" || d.Pclass === selectedClass;

  return sexMatch && classMatch;
});
```

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
    <h2>${avgAge.toFixed(1)}</h2>
    <p>Average age</p>
  </div>

  <div class="card">
    <h2>£${avgFare.toFixed(2)}</h2>
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

## Interactive Passenger Comparison

Choose variables for the X and Y axes, filter by sex and passenger class, and change how passengers are colored.

```js
vl.markCircle({size: 80, opacity: 0.75})
  .data(filteredTitanic)
  .params(
    vl.selectInterval().bind("scales")
  )
  .encode(
    vl.x()
      .fieldQ(xVariable)
      .title(xVariable),

    vl.y()
      .fieldQ(yVariable)
      .title(yVariable),

    colorBy === "Survived"
      ? vl.color()
          .fieldN("Survived")
          .scale({
            domain: [0, 1],
            range: ["red", "green"]
          })
          .title("Survival Status")
      : vl.color()
          .fieldN(colorBy)
          .title(colorBy),

    vl.tooltip([
      "Name",
      "Sex",
      "Age",
      "Fare",
      "Pclass",
      "SibSp",
      "Parch",
      "Survived"
    ])
  )
  .width(800)
  .height(500)
  .render()
```

## Survival Rate by Passenger Class

This bar chart updates with the same filters as the scatterplot.

```js
const classSummary = d3.rollups(
  filteredTitanic,
  v => ({
    passengers: v.length,
    survivalRate: d3.mean(v, d => d.Survived) * 100
  }),
  d => d.Pclass
).map(([Pclass, values]) => ({
  Pclass: `Class ${Pclass}`,
  passengers: values.passengers,
  survivalRate: values.survivalRate
}));
```

```js
vl.markBar()
  .data(classSummary)
  .encode(
    vl.x().fieldN("Pclass").title("Passenger Class"),
    vl.y().fieldQ("survivalRate").title("Survival Rate (%)"),
    vl.tooltip(["Pclass", "passengers", "survivalRate"])
  )
  .width(700)
  .height(350)
  .render()
```

Loaded ${titanic.length} passengers.

# Design Rationale

First of all, I thought that the topic itself would be kind of interesting. I like the movie about the titanic (until it starts sinking) and I thought I could get some good interactions out of it. I wanted to start off with a little boat going across the screen just for something fun. I remember one of the visualizations we looked at had blocks moving across the screen depending on the forces acting on it so I wanted to have something moving across to get a similar aesthetic. I also wanted to do an adjustable scatter plot to make the plot itself more interactive and interesting. You can zoom in and out of the chart and also switch the x and y axis values. The colors correspond to either survived, sex, or passenger class, an you can also filter by sex or passenger class. In order to help guide the user, I added a dashboard that gives some statistics of what is currently selected. I think that this helps give the user more understanding with what is going on and more confidence when exploring the chart. Finally I wanted to make a bar chart below to highlight survival rate. This is the main message I am trying to get across throughout this visualization so I think it is an important inclusion. I also wanted to make sure that when you change the color by, the colors change to clearly indicate that something new is being shown.


# References

Dataset Source:

Titanic Dataset by M Yasser H

https://www.kaggle.com/datasets/yasserh/titanic-dataset

Visualization Inspiration:

CSC 477 course materials, lectures, and in-class examples.

AI Assistance:

OpenAI ChatGPT (GPT-5.5) was used to assist with debugging Observable Framework code, resolving implementation errors, and troubleshooting visualization display issues. All design decisions, interaction choices, and written rationale were created fully by the author.