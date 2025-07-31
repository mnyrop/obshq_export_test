# Final Submission


# NYU Transportation & Mobility Analysis
By: Qahtan Al Jammali qaj201@nyu.edu & Omer Basar ofb2004@nyu.edu

## Introduction


Transportation networks form the lifeblood of modern cities, with university campuses serving as crucial hubs within these mobility systems. At New York University, the diverse community of students, faculty, and staff navigates the complex urban landscape of NYC daily as they commute to and from campus. Our anaylsis of the 2024 transportation survey results examines how the NYU community travels to campus, investigating what factors influence their transportation choices and how considerations such as safety, convenience, cost, and environmental impact shape their daily journeys. By analyzing current commuting patterns across different locations and community roles, this study provides a comprehensive snapshot of urban mobility within the NYU ecosystem, offering valuable insights into the transportation preferences of this dynamic academic community.

## Dataset

The data is the result of the New York University 2024 Transportation Survey conducted by the Office of Sustainability and the Office of Institutional Reserach and Data Integrity. The data comprises of 17,700 responses, 11,929 being students, 1,767 Faculty/Instructors and 4,004 staff and administrators. The survey tackled a multitude of topics, including commute modes, times, and locations. It also delved into non-commute travels such as flights, train trips, car trips, etc. The survey also delved into the reasons behind choosing those travel methods, the time spent on each travel method per day, and the frequency of commute per week. The survey took a respondent about 15 minutes, where they had to answer around 40-50 questions to detail their transit habits which will allow us to explore transit trends at NYU, as well as explain phenomena. Please note data was cleaned, reformattedand anonymized. The data has been anonymized to protect the privacy of NYU community members. Please also note that this data is not to be distrubted or used without approval of the New York University, Office of Sustainability.

```js
import * as d3 from "npm:d3";
import {FileAttachment} from "observablehq:stdlib";
const data2024_v21_csv = FileAttachment("data2024_v21.csv").csv();
const data2024_v21 = d3.csvParse(await FileAttachment("data2024_v21.csv").text());
display(Inputs.table(data2024_v21_csv));
```

## Question 1: Mode (Stacked Bar Chart)

What types of transportation are prevalent among surveyed individuals. Is there a difference between the different affiliations (students, Faculty or Instructor, Staff or Administrator)?

### Rationale & Explanation:

In this first graph, we chose to go with an interactive bar chart. the x-axis shows the different modes, the y-axis shows the NYU population count that uses that mode as part of their commute. 
The bar chart is a stacked bar chart, where green is for student, orange is for Faculty or Instructor, and purple is for Staff or Administrator. 

You can click the different category boxes on the legend to filter out the data based on affiliation type. 

In order to get the exact count you would hover over that stacked bar chart over the stack that you want to know the exact number of. 

The legend allows some interactivity and that is by toggling on and off the different categories, this allows the user to see each category's chart to see what are the top modes they take and what are the ones that don't take as much ... etc
The graph also has smooth transition, and the y-scale scales to the categories chosen.

This graph allows the user to explore the different modes that the NYU community takes as a whole or dig deeper into one or two of the affiliation types, such as seeing what modes people take that are not students, it would show a different story than if we are to only look at students. It seems staff, administrators, faculty and staff prefer Subway, then Rail then Car, meanwhile students chose subway, then the second is walk only and the third is bus. 

It seems from the chart created that NYU as a whole has uses Subway the most, and then Walking, Bus is the third, Rail is fourth Path is fifth, and Shuttle is the sixth most used mode.
The least used mode is Motorcycle, Carpool, Micro and Ferry. 
If we are looking at Students only, you could see that subway and walking are still number 1 and 2, but then Path, Shuttle and Bus take the next , finally ending with Rail.

Meanwhile when you toggle student off and keep faculty, instructors, staff and administrators, you can see that subway remains number 1, but Rail is number 2 and Car is number 3, car is not even in the top 6 for Students. Then you can see Bus, Walking and Bike taken the positions afterwards. 

So even though students form the majority of the campus users, they only form 50% of the car users.

Staff, Faculty, administrators and staff form the minority of NYU community, but they are 50% of car users, and as would be shown in Question 5, car uses are only 3.9% of the community, but form 22% of the transportation emissions.


```js
const affiliations = ["Student", "Faculty or Instructor", "Staff or Administrator"];
const affiliationWeights = new Map([
  ["Student",                4.810294241],
  ["Faculty or Instructor",  3.838143746],
  ["Staff or Administrator", 1.934815185]
]);
  
const data = data2024_v21.map(d => {
  const row = {...d};
  ["walkingTime_v2","subwayTime","busTime","shuttleTime","pathTime",
    "railTime","taxiTime","bikeTime","microTime","motorcycleTime",
    "carTime","carpoolTime","ferryTime","otherTime"]
    .forEach(k => row[k] = +d[k] || 0);
  row.weight = affiliationWeights.get(d.nyuAffiliations) || 0;
  return row;
});
  
const modeTests = {
  "Walk Only": d =>
    d.walkingTime_v2 > 0 &&
    ["subwayTime","busTime","shuttleTime","pathTime",
      "railTime","taxiTime","bikeTime","microTime",
      "motorcycleTime","carTime","carpoolTime",
      "ferryTime","otherTime"].every(k => d[k] === 0),
    Subway:      d => d.subwayTime > 0,
    Bus:         d => d.busTime > 0,
    Shuttle:     d => d.shuttleTime > 0,
    Path:        d => d.pathTime > 0,
    Rail:        d => d.railTime > 0,
    Taxi:        d => d.taxiTime > 0,
    Bike:        d => d.bikeTime > 0,
    Micro:       d => d.microTime > 0,
    Motorcycle:  d => d.motorcycleTime > 0,
    Car:         d => d.carTime > 0,
    Carpool:     d => d.carpoolTime > 0,
    Ferry:       d => d.ferryTime > 0,
    Other:       d => d.otherTime > 0
};
  
let summary = Object.entries(modeTests).map(([mode, test]) => {
  const obj = { mode };
  affiliations.forEach(aff => {
    obj[aff] = data
      .filter(d => d.nyuAffiliations === aff && test(d))
      .reduce((s, d) => s + d.weight, 0);
  });
  return obj;
})
.sort((a, b) =>
  d3.sum(affiliations, k => b[k]) - d3.sum(affiliations, k => a[k])
);
  
const margin = { top: 60, right: 20, bottom: 80, left: 80 },
      W = 900,
      H = 700;
  
const x = d3.scaleBand()
  .domain(summary.map(d => d.mode))
  .range([margin.left, W - margin.right])
  .padding(0.1);
  
const y = d3.scaleLinear()
  .range([H - margin.bottom, margin.top]);
  
const color = d3.scaleOrdinal()
  .domain(affiliations)
  .range(d3.schemeDark2.slice(0, affiliations.length));
  
const container = d3.create("div").style("position","relative");
const tooltip = container.append("div")
  .style("position","absolute")
  .style("pointer-events","none")
  .style("visibility","hidden")
  .style("background","rgba(0,0,0,0.7)")
  .style("color","#fff")
  .style("padding","4px 6px")
  .style("border-radius","4px")
  .style("font-size","12px");
  
const svg = container.append("svg")
  .attr("width", W)
  .attr("height", H);

svg.append("text")
  .attr("class","chart-title")
  .attr("x", W / 2)
  .attr("y", margin.top / 2)
  .attr("text-anchor", "middle")
  .style("font-size", "18px")
  .style("font-weight", "bold")
  .text("Count of People by Mode of Transportation");

const xAxisG = svg.append("g")
  .attr("class","x axis")
  .attr("transform", `translate(0,${H - margin.bottom})`);

const yAxisG = svg.append("g")
  .attr("class","y axis")
  .attr("transform", `translate(${margin.left},0)`);

xAxisG.call(d3.axisBottom(x))
  .selectAll("text")
    .attr("transform","rotate(45)")
    .style("text-anchor","start");

svg.append("text")
  .attr("class","x label")
  .attr("x", W / 2)
  .attr("y", H - margin.bottom + 60)
  .attr("text-anchor","middle")
  .style("font-size","14px")
  .text("Mode");

svg.append("text")
  .attr("class","y label")
  .attr("transform","rotate(-90)")
  .attr("x", -H / 2)
  .attr("y", margin.left - 50)
  .attr("text-anchor","middle")
  .style("font-size","14px")
  .text("Count");
  
const legend = svg.append("g")
  .attr("transform", `translate(${W - margin.right - 200},${margin.top})`);

legend.append("text")
  .attr("class", "legend-title")
  .attr("x", 0)
  .attr("y", 0)
  .style("font-size", "14px")
  .style("font-weight", "bold")
  .text("Click boxes to filter");

let active = new Set(affiliations);
affiliations.forEach((aff, i) => {
  const g = legend.append("g")
    .attr("transform", `translate(0,${i * 20 + 20})`)
    .style("cursor", "pointer")
    .on("click", () => {
      active.has(aff) ? active.delete(aff) : active.add(aff);
      legend.selectAll("rect")
        .style("opacity", d => active.has(d) ? 1 : 0.3);
      draw();
    });

  g.append("rect")
    .datum(aff)
    .attr("width", 12)
    .attr("height", 12)
    .attr("fill", color(aff))
    .style("opacity", 1);

  g.append("text")
    .attr("x", 18)
    .attr("y", 6)
    .attr("dy", "0.35em")
    .text(aff);
});
  
  function draw() {
    const t = svg.transition().duration(600);
    const keys = Array.from(active);
    const series = d3.stack().keys(keys)(summary);
  
    const flat = series.flatMap(serie =>
      serie.map(([y0, y1], i) => ({
        mode:        summary[i].mode,
        affiliation: serie.key,
        y0, y1,
        value:       summary[i][serie.key]
      }))
    );
  
    y.domain([0, d3.max(flat, d => d.y1)]).nice();
    svg.select(".y.axis")
      .transition(t)
      .call(d3.axisLeft(y));
  
    const bars = svg.selectAll("rect.stack")
      .data(flat, d => d.mode + "|" + d.affiliation);
  
    bars.exit()
      .transition(t)
        .attr("y", y(0))
        .attr("height", 0)
        .remove();
  
    const barsEnter = bars.enter()
      .append("rect")
        .classed("stack", true)
        .attr("x", d => x(d.mode))
        .attr("width", x.bandwidth())
        .attr("y", y(0))
        .attr("height", 0)
        .attr("fill", d => color(d.affiliation))
        .on("mouseover", (e, d) => {
          tooltip
            .html(`${d.affiliation}: ${d.value.toLocaleString()}`)
            .style("visibility","visible");
        })
        .on("mousemove", e => {
          const [mx, my] = d3.pointer(e, container.node());
          tooltip.style("left", `${mx + 5}px`)
                 .style("top", `${my + 5}px`);
        })
        .on("mouseout", () => tooltip.style("visibility","hidden"));
  
    barsEnter.merge(bars)
      .transition(t)
        .attr("x", d => x(d.mode))
        .attr("width", x.bandwidth())
        .attr("y", d => y(d.y1))
        .attr("height", d => y(d.y0) - y(d.y1));
  }

  draw()
  
  display(container.node());     

```

## Question 2: Time (Line Charts)
What time do NYU community members typically arrive on campus and when do they depart? How does it fluctuate throughout the week? What is the total population number at different campuses throughout the day and throughout the week?

### Rationale & Explanation:

For our second chart, we decided to go with line charts that could be toggled to switch between modes consisting of arrivals times, departure times, and  overall populations on NYU designated locations. We have the x axis dedicated to the time bins and the y axis is related to the amount of people. We have multiple lines for each chart color coded to represent the various days of the week, which can also be toggled in order to make easier comparisons. In addition we also included toggles for affiliation with NYU as well as the location that people are commuting to.

Looking at the arrivals chart, we can see that majority people arrive to their destination between 9AM to 9:59AM, while most depart at 5PM to 5:59PM. We also noticed that most of the foot traffic on campus occurs around the afternoon and that the most popular days are Monday through Thursday. People on the weekends tend to arrive at a later time than the weekdays.

For arrivals the peak is on Wednesday with 14,000 arrivals from 9AM to 9:59AM, which is basically rush hour. For departures, the peak is on Tuesday with 11,000 arrivals from 5PM to 5:59PM. Prime time student populations are between 11AM and 4PM during the week, peaking at about 37,000 people, meaning it's a great time to host events.

```js  
// Define required fields (must have valid values) - only essential ones
let required = [
  "ResponseId"
];

// Define arrival/departure fields that need valid time values (not "varies")
const timeScheduleFields = [
  "MonArrival", "TueArrival", "WedArrival", "ThuArrival", "FriArrival", "SatArrival", "SunArrival",
  "monDeparture", "tueDeparture", "wedDeparture", "thuDeparture", "friDeparture", "satDeparture", "sunDeparture"
];

// Define time fields that can be converted to 0 if invalid
const timeFields = [
  "shuttleTime",
  "walkingTime_v2",
  "busTime",
  "subwayTime",
  "pathTime",
  "railTime",
  "taxiTime",
  "bikeTime",
  "microTime",
  "motorcycleTime",
  "carTime",
  "carpoolTime",
  "ferryTime",
  "otherTime"
];

// Helper function to check if a value is effectively empty/missing
const isEmptyValue = (value) => {
  if (value == null) return true;
  const str = String(value).trim().toLowerCase();
  return str === "" || 
          str === "please select an option" || 
          str === "nan" ||
          str === "no usual time/it varies" ||
          str === "no usual departure time/it varies";
};

// Helper function to clean time values - convert invalid to 0
const cleanTimeValue = (value) => {
  if (isEmptyValue(value) || isNaN(value)) {
    return 0;
  }
  return +value; // convert to number
};

// Helper function to clean required values - convert invalid to default
const cleanRequiredValue = (value, defaultValue = "") => {
  if (isEmptyValue(value)) {
    return defaultValue;
  }
  return value;
};

const cleaned = data2024_v21
  .filter(d => {
    // Check required fields
    const hasRequiredFields = required.every(field => {
      const v = d[field];
      return v != null && String(v).trim() !== "";
    });
    
    // Check that exactly one of travelDaysPerWeek or travelDaysPerMonth is filled
    const hasWeekly = !isEmptyValue(d.travelDaysPerWeek);
    const hasMonthly = !isEmptyValue(d.travelDaysPerMonth);
    const hasValidTravelData = (hasWeekly && !hasMonthly) || (!hasWeekly && hasMonthly);
    
    // Check that at least one day has valid arrival AND departure times (not "varies")
    const hasValidSchedule = timeScheduleFields.some((field, index) => {
      if (index < 7) { // arrival fields
        const arrivalField = field;
        const departureField = timeScheduleFields[index + 7];
        return !isEmptyValue(d[arrivalField]) && !isEmptyValue(d[departureField]);
      }
      return false; // skip departure fields in this loop since we check them with arrivals
    });
    
    return hasRequiredFields && hasValidTravelData && hasValidSchedule;
  })
  .map(d => ({
    ResponseId:         d.ResponseId,
    nyuAffiliations:    !isEmptyValue(d.nyuAffiliations) ? d.nyuAffiliations : "Unknown",
    weight:             !isEmptyValue(d.weight) && !isNaN(+d.weight) ? +d.weight : 1,
    primaryLocation:    !isEmptyValue(d.primaryLocation) ? d.primaryLocation : "Unknown",
    MonArrival:         cleanRequiredValue(d.MonArrival),
    TueArrival:         cleanRequiredValue(d.TueArrival),
    WedArrival:         cleanRequiredValue(d.WedArrival),
    ThuArrival:         cleanRequiredValue(d.ThuArrival),
    FriArrival:         cleanRequiredValue(d.FriArrival),
    SatArrival:         cleanRequiredValue(d.SatArrival),
    SunArrival:         cleanRequiredValue(d.SunArrival),
    monDeparture:       cleanRequiredValue(d.monDeparture),
    tueDeparture:       cleanRequiredValue(d.tueDeparture),
    wedDeparture:       cleanRequiredValue(d.wedDeparture),
    thuDeparture:       cleanRequiredValue(d.thuDeparture),
    friDeparture:       cleanRequiredValue(d.friDeparture),
    satDeparture:       cleanRequiredValue(d.satDeparture),
    sunDeparture:       cleanRequiredValue(d.sunDeparture),
    journeyTime:        cleanRequiredValue(d.journeyTime),
    travelFrequency:    !isEmptyValue(d.travelDaysPerWeek) ? `${d.travelDaysPerWeek} (per week)` : 
                        !isEmptyValue(d.travelDaysPerMonth) ? `${d.travelDaysPerMonth} (per month)` : null,
    shuttle:            cleanTimeValue(d.shuttleTime),
    walking:            cleanTimeValue(d.walkingTime_v2),
    bus:                cleanTimeValue(d.busTime),
    subway:             cleanTimeValue(d.subwayTime),
    path:               cleanTimeValue(d.pathTime),
    rail:               cleanTimeValue(d.railTime),
    taxi:               cleanTimeValue(d.taxiTime),
    bike:               cleanTimeValue(d.bikeTime),
    micro:              cleanTimeValue(d.microTime),
    motorcycle:         cleanTimeValue(d.motorcycleTime),
    car:                cleanTimeValue(d.carTime),
    carpool:            cleanTimeValue(d.carpoolTime),
    ferry:              cleanTimeValue(d.ferryTime),
    other:              cleanTimeValue(d.otherTime)
  }));

display(cleaned);
const extracted = cleaned;
```

```js
function normalize(bin) {
  return bin
    .trim()
    .replace(/\u2013/g, "-")       // turn en-dash into hyphen
    .replace(/\s+/g, " ")          // collapse multiple spaces
}
```

```js
  const days = [
    {abbr: "Mon", full: "Monday",    arrivalKey: "MonArrival",    depKey: "monDeparture"},
    {abbr: "Tue", full: "Tuesday",   arrivalKey: "TueArrival",    depKey: "tueDeparture"},
    {abbr: "Wed", full: "Wednesday", arrivalKey: "WedArrival",    depKey: "wedDeparture"},
    {abbr: "Thu", full: "Thursday",  arrivalKey: "ThuArrival",    depKey: "thuDeparture"},
    {abbr: "Fri", full: "Friday",    arrivalKey: "FriArrival",    depKey: "friDeparture"},
    {abbr: "Sat", full: "Saturday",  arrivalKey: "SatArrival",    depKey: "satDeparture"},
    {abbr: "Sun", full: "Sunday",    arrivalKey: "SunArrival",    depKey: "sunDeparture"}
  ];

  const timeBins = [
    "Midnight - 5:59 AM","6 AM - 6:59 AM","7 AM - 7:59 AM","8 AM - 8:59 AM",
    "9 AM - 9:59 AM","10 AM - 10:59 AM","11 AM - 11:59 AM","12 PM - 12:59 PM",
    "1 PM - 1:59 PM","2 PM - 2:59 PM","3 PM - 3:59 PM","4 PM - 4:59 PM",
    "5 PM - 5:59 PM","6 PM - 6:59 PM","7 PM - 7:59 PM","8 PM - 8:59 PM",
    "9 PM - 9:59 PM","10 PM - 10:59 PM","11 PM - 11:59 PM"
  ];

  const affiliations = Array.from(new Set(extracted.map(d => d.nyuAffiliations))).sort();
  const locations    = Array.from(new Set(extracted.map(d => d.primaryLocation))).sort();
  
  // Get unique travel frequency values (combining weekly and monthly)
  const travelFrequencyOptions = Array.from(new Set(
    extracted
      .map(d => d.travelFrequency)
      .filter(d => d != null)
  )).sort();
  
  // Add "Unknown" option for null values (though there shouldn't be any due to filtering)
  if (extracted.some(d => d.travelFrequency == null)) {
    travelFrequencyOptions.push("Unknown");
  }
  
  // Transportation modes
  const transportationModes = [
    {key: "shuttle", label: "Shuttle"},
    {key: "walking", label: "Walking"},
    {key: "bus", label: "Bus"},
    {key: "subway", label: "Subway"},
    {key: "path", label: "PATH"},
    {key: "rail", label: "Rail"},
    {key: "taxi", label: "Taxi"},
    {key: "bike", label: "Bike"},
    {key: "micro", label: "Micro-mobility"},
    {key: "motorcycle", label: "Motorcycle"},
    {key: "car", label: "Car"},
    {key: "carpool", label: "Carpool"},
    {key: "ferry", label: "Ferry"},
    {key: "other", label: "Other"}
  ];

  // UI controls
  const chartType = Inputs.select(
    ["Arrival times","Departure times","On-campus population"],
    {label: "Chart:", value: "Arrival times"}
  );
  const daySelector = Inputs.checkbox(
    days.map(d => d.full),
    {label: "Days:", value: days.map(d => d.full), columns: 7}
  );
  const affiliationSelector = Inputs.checkbox(
    affiliations,
    {label: "Affiliations:", value: affiliations, columns: 3}
  );
  const locationSelector = Inputs.checkbox(
    locations,
    {label: "Locations:", value: locations, columns: 3}
  );
  const travelFrequencySelector = Inputs.checkbox(
    travelFrequencyOptions,
    {label: "Travel frequency:", value: travelFrequencyOptions, columns: 4}
  );
  const transportationSelector = Inputs.checkbox(
    transportationModes.map(t => t.label),
    {label: "Transportation modes:", value: transportationModes.map(t => t.label), columns: 4}
  );
  const transportationLogic = Inputs.select(
    ["OR", "AND", "ONLY"],
    {label: "Transportation filter logic:", value: "OR"}
  );

  // container for chart
  const container = html`<div></div>`;

  function render() {
    container.replaceChildren();

    // filter by affiliation, location, and travel frequency
    const selAff = affiliationSelector.value;
    const selLoc = locationSelector.value;
    const selTravelFreq = travelFrequencySelector.value;
    const selTransportLabels = transportationSelector.value;
    const transportMode = transportationLogic.value;
    
    // Convert transport labels back to keys
    const selTransportKeys = selTransportLabels.map(label => 
      transportationModes.find(t => t.label === label)?.key
    ).filter(Boolean);

    let filtered = extracted.filter(d => {
      const affMatch = selAff.includes(d.nyuAffiliations);
      const locMatch = selLoc.includes(d.primaryLocation);
      
      // Handle travel frequency filtering with "Unknown" option
      let travelFreqMatch = false;
      if (d.travelFrequency == null) {
        // For null values, check if "Unknown" is selected
        travelFreqMatch = selTravelFreq.includes("Unknown");
      } else {
        // For actual values, check if the value is selected
        travelFreqMatch = selTravelFreq.includes(d.travelFrequency);
      }
      
      return affMatch && locMatch && travelFreqMatch;
    });

    // Apply transportation filtering
    if (selTransportKeys.length > 0) {
      filtered = filtered.filter(d => {
        if (transportMode === "OR") {
          // OR logic: user must use at least one of the selected modes (time > 0)
          return selTransportKeys.some(key => d[key] > 0);
        } else if (transportMode === "AND") {
          // AND logic: user must use ALL selected modes (time > 0 for all)
          return selTransportKeys.every(key => d[key] > 0);
        } else if (transportMode === "ONLY") {
          // ONLY logic: user must use EXACTLY the selected modes and no others
          const allTransportKeys = transportationModes.map(t => t.key);
          
          // Check that all selected modes are used (time > 0)
          const usesAllSelected = selTransportKeys.every(key => d[key] > 0);
          
          // Check that no unselected modes are used (time = 0)
          const unselectedKeys = allTransportKeys.filter(key => !selTransportKeys.includes(key));
          const usesOnlySelected = unselectedKeys.every(key => d[key] === 0);
          
          return usesAllSelected && usesOnlySelected;
        }
      });
    } else {
      // If no transportation modes selected, show no data
      filtered = [];
    }

    // compute weighted counts per bin
    // prepare flat arrays with {bin, weight}
    function weightedEvents(key) {
      return filtered.flatMap(d => {
        const bin = normalize(d[key]);
        return bin ? [{bin, weight: d.weight}] : [];
      });
    }

    const arrivalCountsByDay = new Map(days.map(day => [
      day.abbr,
      d3.rollup(
        weightedEvents(day.arrivalKey),
        v => d3.sum(v, d => d.weight),
        d => d.bin
      )
    ]));

    const departureCountsByDay = new Map(days.map(day => [
      day.abbr,
      d3.rollup(
        weightedEvents(day.depKey),
        v => d3.sum(v, d => d.weight),
        d => d.bin
      )
    ]));

    // population: running sum of weighted arrivals minus departures per bin
    const populationCountsByDay = new Map(days.map(day => {
      const aMap = arrivalCountsByDay.get(day.abbr);
      const dMap = departureCountsByDay.get(day.abbr);
      let cum = 0;
      const series = timeBins.map(bin => {
        const a = aMap.get(bin) || 0;
        const e = dMap.get(bin) || 0;
        cum = Math.max(0, cum + a - e);
        return {bin, count: cum};
      });
      return [day.abbr, series];
    }));

    // selected days
    const selDays = daySelector.value;
    const selAbbrs = selDays.map(full => days.find(d => d.full === full).abbr);

    // build series for chart type
    let series, yLabel;
    if (chartType.value === "Arrival times") {
      yLabel = "Arrivals";
      series = selAbbrs.map(abbr => ({
        key: abbr,
        values: timeBins.map(bin => ({bin, count: arrivalCountsByDay.get(abbr).get(bin) || 0}))
      }));
    } else if (chartType.value === "Departure times") {
      yLabel = "Departures";
      series = selAbbrs.map(abbr => ({
        key: abbr,
        values: timeBins.map(bin => ({bin, count: departureCountsByDay.get(abbr).get(bin) || 0}))
      }));
    } else {
      yLabel = "Population";
      series = selAbbrs.map(abbr => ({
        key: abbr,
        values: populationCountsByDay.get(abbr)
      }));
    }

    if (!series.length || filtered.length === 0) {
      container.innerHTML = "<strong>Please select at least one day, affiliation, location, travel frequency, and transportation mode.</strong>";
      return;
    }

    // draw chart
    const M = {top: 50, right: 160, bottom: 80, left: 70},
          W = 900 - M.left - M.right,
          H = 500 - M.top - M.bottom;

    const svg = d3.create("svg")
        .attr("viewBox", [0, 0, W + M.left + M.right, H + M.top + M.bottom]);

    // Chart title
    svg.append("text")
       .attr("x", (W+M.left+M.right)/2)
       .attr("y", 25)
       .attr("text-anchor", "middle")
       .attr("font-size", "16px")
       .attr("font-weight", "bold")
       .text(chartType.value + " by Time Bin");

    const x = d3.scalePoint()
        .domain(timeBins)
        .range([M.left, M.left + W])
        .padding(0.5);
    const y = d3.scaleLinear()
        .domain([0, d3.max(series, s => d3.max(s.values, d => d.count))]).nice()
        .range([M.top + H, M.top]);

    const color = d3.scaleOrdinal(d3.schemeCategory10)
        .domain(series.map(s => s.key));

    // axes
    svg.append("g")
       .attr("transform", `translate(0, ${M.top+H})`)
       .call(d3.axisBottom(x).tickFormat(d => d))
       .selectAll("text")
         .attr("text-anchor", "end")
         .attr("transform", "rotate(-40)");
    svg.append("g")
       .attr("transform", `translate(${M.left}, 0)`)
       .call(d3.axisLeft(y));

    // axis labels
    svg.append("text")
       .attr("x", M.left + W/2)
       .attr("y", M.top + H + 80)
       .attr("text-anchor", "middle")
       .attr("font-weight", "bold")
       .text("Time Bin");
    svg.append("text")
       .attr("transform", "rotate(-90)")
       .attr("x", -(M.top + H/2))
       .attr("y", M.left - 50)
       .attr("text-anchor", "middle")
       .attr("font-weight", "bold")
       .text(yLabel);

    // lines
    const line = d3.line()
        .x(d => x(d.bin))
        .y(d => y(d.count));
    series.forEach(s => {
      svg.append("path")
         .datum(s.values)
         .attr("fill", "none")
         .attr("stroke", color(s.key))
         .attr("stroke-width", 1.5)
         .attr("d", line);
    });

    // legend
    const legend = svg.append("g")
        .attr("transform", `translate(${M.left + W + 20}, ${M.top})`);
    legend.append("text")
       .attr("x", 0)
       .attr("y", -10)
       .attr("font-weight", "bold")
       .text("Days");
    series.forEach((s, i) => {
      const y0 = i * 20;
      legend.append("rect")
        .attr("x", 0)
        .attr("y", y0)
        .attr("width", 12)
        .attr("height", 12)
        .attr("fill", color(s.key));
      legend.append("text")
        .attr("x", 18)
        .attr("y", y0 + 10)
        .text(days.find(d => d.abbr === s.key).full);
    });

    container.append(svg.node());
  }

  // attach listeners
  chartType.addEventListener("input", render);
  daySelector.addEventListener("input", render);
  affiliationSelector.addEventListener("input", render);
  locationSelector.addEventListener("input", render);
  travelFrequencySelector.addEventListener("input", render);
  transportationSelector.addEventListener("input", render);
  transportationLogic.addEventListener("input", render);
  render();

  // layout selectors on separate lines
  display(html`
    <div>${chartType}</div>
    <div>${daySelector}</div>
    <div>${affiliationSelector}</div>
    <div>${locationSelector}</div>
    <div>${travelFrequencySelector}</div>
    <div>${transportationSelector}</div>
    <div>${transportationLogic}</div>
    ${container}
  `);
```

## Question 3: Distance (Choropleth Map)

How far do NYU community members travel every day? Where do they travel from? where do they travel to?

### Rationale & Explanation:

Here we created a Choropleth Map, since we had zip code departures, it made sense to go that route than the heat map, which we tested out on the side, but the latitude and longitude for the zip codes made it look artifical and gave a false sense of accuracy that we did not have. 

The creation of the choropleth map required the download of multiple state geoJSONs, then the combining of those geoJSONs and then the trimming of that geoJSON file to limit it to 125 mile radius from NYU. the pre-processing was done using Python and its geo processing libraries.

The thought process behind this is to create the choropleth map, and then create filters as we see fit to answer our quetsions.
The scale was creted with a with as a power scale with power of 0.2, to add more sensitivity towards the smaller numbers, and less towards the few zip codes that had a great number of students living on campus. 

Looking at the choropleth map you could see where NYU community members are traling from to get to NYU, which shows a sense of how far they are coming from. You can filter based on where they are going whether it is Washington square Campus, Brooklyn Campus (Mainly Tandon), Union Square, Health Corridor, Upper East Side, or the ones that are not affiliated with one or any campus. 
You are able to filter based on the affiliation type, such as whether they are a student, a faculty or insturctor, or staff or administrator. 

Based on the questions above you can filter and see the departure zip codes based on all the different ways you want to filter the data.

Some of the inferences that can be noticed from the choropleth map are the differences between the departure locations for Washington Square Campus versus the Brooklyn Campus. For the Washington Square Campus, you can see that most people live in midtown and nearby Brooklyn, as well as some accessible zip codes via the PATH train from New Jersey. Meanwhile for the Brooklyn Campus departure locations they are more likely to be in downtown Brooklyn, and there is a good concentration of community members in Bayridge and the neighboroods above it. There are still a good amount in downtown manhattan, and PATH accessible New Jersey. 

When comparing students versus staff and faculty. We noticed that more students lived on or close to campus, meanwhile staff and faculty tend to be more evenly distributed over a larger radius. Which makes sense due to first year students mainly living on campus, and when coming from out of state, we expect students to want to live close to campus, while the staff and faculty tend to be full time New Yorkers that live further away from campus.

```js
radius_125 = FileAttachment("trimmed.geojson").json()
```

```js
width = 960
```

```js
height = 600
```

```js
projection = d3.geoMercator()
  .fitExtent(
    [[20, 20], [width - 20, height - 20]],
    radius_125
  )
```

```js
path = d3.geoPath(projection)
```

```js
radius_125Map2 = {
  const container = html`<div style="text-align:center"></div>`;

  container.append(html`
    <div style="
      font-size: 1.5em;
      font-weight: bold;
      margin-bottom: 0.5em;
    ">
      Departure Zip Code Choropleth Map
    </div>
  `);

  const svg = d3.create("svg")
    .attr("viewBox", [0, 0, width, height])
    .style("width", "100%")
    .style("height", "auto")
    .style("cursor", "grab");

  const mapG = svg.append("g")
    .attr("class", "map-layer");

  const weightByZip = d3.rollup(
    filteredData,
    v => d3.sum(v, d => +d.weight),
    d => String(d.departureZipCode).padStart(5, "0")
  );

  const features = radius_125.features.map(feat => {
    const raw = feat.properties.ZCTA5CE10 ?? feat.properties.ZIP ?? feat.id;
    feat.properties._zip5 = String(raw).padStart(5, "0");
    return feat;
  });

  const maxWeight = d3.max(weightByZip.values());
  const color = d3.scaleSequentialPow()
    .domain([0, maxWeight])
    .exponent(0.2)
    .interpolator(d3.interpolatePurples);

  mapG.selectAll("path")
    .data(features)
    .join("path")
      .attr("d", path)
      .attr("fill", d => {
        const w = weightByZip.get(d.properties._zip5) || 0;
        return w > 0 ? color(w) : "#eee";
      })
      .attr("stroke", "#555")
      .attr("stroke-width", 0.05)
    .append("title")
      .text(d => {
        const w = Math.round(weightByZip.get(d.properties._zip5) || 0);
        return `${d.properties._zip5}\nTotal extrapolated responses: ${w}`;
      });


  const zoom = d3.zoom()
    .scaleExtent([1, 60])
    .on("zoom", event => mapG.attr("transform", event.transform));

  svg.call(zoom);
  const initialScale = 7;
  const center = [width / 2 + 25, height / 2];
  svg.call(zoom.transform, d3.zoomIdentity
    .translate(center[0], center[1])
    .scale(initialScale)
    .translate(-center[0], -center[1]));

  container.append(svg.node());

  const legendWidth  = 300;
  const legendHeight = 10;
  const margin       = { top: 20, right: 20, bottom: 20, left: 20 };

  const legendSvg = d3.create("svg")
    .attr("viewBox", [0, 0, legendWidth + margin.left + margin.right, legendHeight + margin.top + margin.bottom])
    .style("width", `${legendWidth + margin.left + margin.right}px`)
    .style("margin-top", "1em");

  const defs = legendSvg.append("defs")
    .append("linearGradient")
      .attr("id", "legend-gradient")
      .attr("x1", "0%").attr("x2", "100%")
      .attr("y1", "0%").attr("y2", "0%");

  defs.selectAll("stop")
    .data(d3.range(0, 1.01, 0.1))
    .enter().append("stop")
      .attr("offset", d => `${d * 100}%`)
      .attr("stop-color", d => color(maxWeight * d));

  const g = legendSvg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  g.append("rect")
    .attr("width", legendWidth)
    .attr("height", legendHeight)
    .style("fill", "url(#legend-gradient)");

  const legendScale = d3.scaleSequentialPow()
    .domain([0, maxWeight])
    .range([0, legendWidth]);

  g.append("g")
    .attr("transform", `translate(0,${legendHeight})`)
    .call(d3.axisBottom(legendScale).ticks(5).tickFormat(d3.format(",.0f")));

  g.append("text")
    .attr("x", legendWidth / 2)
    .attr("y", -5)
    .attr("text-anchor", "middle")
    .style("font-size", "10px")
    .style("font-family", "sans-serif")
    .text("Total Extrapolated Responses");

  container.append(legendSvg.node());

  return container;
}

```

```js
viewof selectedModeFilter = Inputs.radio(
  ["Any of selected", "Only these"],
  {
    label: "Mode‐filter type:",
    value: "Any of selected",
    columns: 1
  }
);
```

```js
viewof selectedModes = Inputs.checkbox(
  all_modes,
  {
    label: "Filter by Transport Mode",
    // default to having them all checked
    value: all_modes,
    // lay them out in columns so it doesn’t get too long
    columns: 3
  }
);
```

```js
viewof selectedLocations = Inputs.checkbox(
  locations,
  {
    label: "Filter by Primary Location",
    value: locations   
  }
);
```

```js
viewof selectedAffiliations = Inputs.checkbox(
  affiliations,
  {
    label: "Filter by Affiliation",
    value: affiliations   
  }
);
```

## Question 4: Carbon Emissions (Waffle Chart)

What are the carbon emissions from the way community members are traveling in 2024? How relative are the carbon emissions to the population that uses a specific method?

### Rationale & Explanation:

For Question 4, We decided to go with a waffle chart to display all of the carbon emissions. We went with waffle because it allows for an easy way to quantify the amount of carbon emissions created by the NYU community and compare based on different modes of transportation. For the waffle chart, each cell/square will represent 20 tons of carbon emission. The cells are color coded based on two factors. The first factor is the type of transportation, which determines the hue of color. The second factor is that shade is determined by the the subtype for transportation. Here we have trains grouped under blue, shared vehicles under green, and individual vehicles under red. Micromobility gets its own color. Walking and biking are not shown on the waffle chart because they are carbon neutral forms of transportation, meaning they don't produce any carbon emissions.

Upon hovering each group of cells, it will display a pie chart that shows the statistics for a given group. For example, the subway which is 45.9% of the population accounts for 24.1% of total emissions.

You can see the overall emissions on the waffle, and the relativity to the population by hovering over the waffle cells to see the pie chart. Just looking at the chart, you can see that a good majority of the carbon emissions is done by those who use cars, upon hovering you will see they only account for 4% of the overall population, which is a huge carbon footprint. In contrast you see the subway occupies a similar amount of carbon, however it is used by 46% of the population, a whopping 11.5x difference in population, meaning they give off less of a carbon footprint per capita.

```js
url5_1 = FileAttachment("carbon_emissions_table_v2.csv").url()
```

```js
url5_2 = FileAttachment("pct_table_v2.csv").url()
```

```js
data5 = ((await d3.csv(url5_1, d => ({
  mode: d.Mode,
  emissions: +d["Emissions (tCO2e)"].replace(/,/g, "")
})))
.slice(0, 15))
.sort((a, b) => a.emissions - b.emissions)
.filter(d => d.emissions > 0)
```

```js
pctData = ((await d3.csv(url5_2, d => ({
  mode: d.Mode === "Walk Only" ? "Walking" : 
        d.Mode === "Classic bike" ? "Bike" : 
        d.Mode === "Electric bike" ? "E-Bike" : 
        d.Mode,
  percent: +d.Percent
}))));
```

```js
totalEmissions = d3.sum(data5, d => d.emissions)
```

```js
{
  // Vars
  const unit        = 20;    // 1 square = 20 tCO₂e
  const cellSize    = 15;
  const legendRect  = 10;    
  const legendGap   = 3;     
  const margin      = 20;
  const pieRadius   = 50;    
  const titleSpace  = 40;
  const pieYOffset  = 0;    

  // Pie layout & arc
  const pieGen = d3.pie().value(d => d.percent).sort(null);
  const arcGen = d3.arc().innerRadius(0).outerRadius(pieRadius);
  const pieData = pieGen(pctData);

  // Updated category & color logic to include E-Bike in micromobility
  const groupMap = {
    Micro:      "micromobility",
    "E-Bike":   "micromobility",
    Subway:     "train", 
    Rail:       "train", 
    Path:       "train",
    Bus:        "shared", 
    Shuttle:    "shared", 
    Ferry:      "shared", 
    Carpool:    "shared",
    Car:        "individual", 
    Taxi:       "individual", 
    Motorcycle: "individual",
    Other:      "individual"  
  };
  
  const groupOrder = ["micromobility","train","shared","individual"];
  const modesByGroup = Object.fromEntries(groupOrder.map(g=>[g,[]]));
  
  // Map mode names for consistency
  const mappedData5 = data5.map(d => ({
    ...d,
    mode: d.mode === "Walk Only" ? "Walking" : 
          d.mode === "Classic bike" ? "Bike" : 
          d.mode === "Electric bike" ? "E-Bike" : 
          d.mode
  }));
  
  mappedData5.forEach(d => {
    const g = groupMap[d.mode];
    if (g && !modesByGroup[g].includes(d.mode)) modesByGroup[g].push(d.mode);
  });

  // Base colors for emissions modes
  const baseModes = mappedData5.map(d=>d.mode);
  const baseColors = mappedData5.map(d => {
    const g    = groupMap[d.mode],
          list = modesByGroup[g],
          t    = (list.indexOf(d.mode)+1)/(list.length+1);
    return g==="micromobility" ? d3.interpolatePurples(t)
         : g==="train"         ? d3.interpolateBlues(t)
         : g==="shared"        ? d3.interpolateGreens(t)
         : g==="individual"    ? d3.interpolateReds(t)
         : "#ccc";
  });

  // Carbon-neutral modes
  const neutralModes = ["Walking","Bike"];
  const neutralColors = ["#FFD700","#FFEA00"];

  // Combined color scale
  const color = d3.scaleOrdinal()
    .domain(baseModes.concat(neutralModes))
    .range(baseColors.concat(neutralColors));

  // Build waffle cells
  const counts = groupOrder.flatMap(g =>
    mappedData5
      .filter(d => groupMap[d.mode] === g)
      .map(d => ({ 
        mode: d.mode, 
        count: Math.max(1, Math.round(d.emissions / unit))
      }))
  );
  const cells = counts.flatMap(d =>
    Array.from({ length: d.count }, () => ({ mode: d.mode }))
  );

  // Grid dimensions
  const total = cells.length,
        cols  = Math.ceil(Math.sqrt(total)),
        rows  = Math.ceil(total/cols),
        baseW = cols * cellSize,
        baseH = rows * cellSize;

  // Create SVG
  let svg = d3.create("svg")
      .attr("width",  baseW + margin*2 + pieRadius*2 + margin + 200)
      .attr("height", titleSpace + Math.max(baseH, 300));

  // Title + hover note
  svg.append("text")
    .attr("x", baseW/2 + margin).attr("y", 18)
    .attr("text-anchor","middle").attr("font-size",18).attr("font-weight","bold")
    .text("2024 Transportation Emissions Waffle Chart");
  svg.append("text")
    .attr("x", baseW/2 + margin).attr("y", 34)
    .attr("text-anchor","middle").attr("font-size",12)
    .text("Hover to see statistics");

  // Chart group
  const chartG = svg.append("g")
    .attr("transform", `translate(${margin}, ${titleSpace})`);

  // Draw waffle cells
  chartG.selectAll("rect.cell")
    .data(cells)
    .join("rect")
      .attr("class","cell")
      .attr("width",  cellSize-1)
      .attr("height", cellSize-1)
      .attr("x", (_,i) => (i % cols) * cellSize)
      .attr("y", (_,i) => (rows - 1 - Math.floor(i/cols)) * cellSize)
      .attr("fill", d => color(d.mode));

  // Legend
  const legendX = baseW + margin*2;
  const legend = chartG.append("g").attr("transform", `translate(${legendX}, 0)`);
  let legendY = 0;

  // Note above legend
  legend.append("text")
    .attr("x",0).attr("y",legendY+legendRect)
    .attr("font-weight","bold").attr("font-size",12)
    .text("Note: 1 square ≈ 20 tCO₂e");
  legendY += legendRect + legendGap + 2;

  // Emissions groups legend
  groupOrder.forEach(group => {
    legend.append("text")
      .attr("x",0).attr("y",legendY+legendRect)
      .attr("font-weight","bold")
      .text(group.charAt(0).toUpperCase() + group.slice(1));
    legendY += legendRect + legendGap;

    modesByGroup[group].forEach(mode => {
      const rec = mappedData5.find(d=>d.mode===mode);
      if (rec) {
        legend.append("rect")
          .attr("width", legendRect).attr("height", legendRect)
          .attr("y", legendY).attr("fill", color(mode));
        legend.append("text")
          .attr("x", legendRect+legendGap)
          .attr("y", legendY+legendRect-1)
          .attr("font-size",10)  
          .text(`${mode}: ${rec.emissions.toLocaleString()} tCO₂e`);
        legendY += legendRect + legendGap;
      }
    });
    legendY += legendGap; 
  });

  // Carbon-neutral legend
  legend.append("text")
    .attr("x",0).attr("y",legendY+legendRect)
    .attr("font-weight","bold")
    .text("Carbon Neutral");
  legendY += legendRect + legendGap;

  neutralModes.forEach(mode => {
    legend.append("rect")
      .attr("width", legendRect).attr("height", legendRect)
      .attr("y", legendY)
      .attr("fill", color(mode));
    legend.append("text")
      .attr("x", legendRect+legendGap)
      .attr("y", legendY+legendRect-1) 
      .attr("font-size",10)  
      .text(`${mode}: 0 tCO₂e`);
    legendY += legendRect + legendGap;
  });

  // Pie container
  const pieGroup = chartG.append("g")
    .attr("transform", `translate(${legendX + pieRadius}, ${legendY + margin + pieRadius + pieYOffset})`);

  // Pie subtitle
  pieGroup.append("text")
    .attr("text-anchor","middle")
    .attr("y", -pieRadius - 8) 
    .attr("font-size",11)  
    .attr("font-weight","bold")
    .text("Population distribution of carbon");

  // Draw default gray pie
  pieGroup.selectAll("path")
    .data(pieData)
    .join("path")
      .attr("d", arcGen)
      .attr("fill", "#ddd")
      .attr("stroke", "#fff")
      .on("mouseenter", (event,d) => highlightMode(d.data.mode))
      .on("mouseleave", resetHighlight);

  // Hover logic
  let currentMode = null;
  svg.on("mousemove", (event) => {
    const [x,y] = d3.pointer(event, chartG.node());
    if (x>=0 && x<baseW && y>=0 && y<baseH) {
      const col = Math.floor(x / cellSize);
      const row = rows - 1 - Math.floor(y / cellSize);
      const idx = row * cols + col;
      if (cells[idx]) {
        const mode = cells[idx].mode;
        if (mode !== currentMode) {
          currentMode = mode;
          highlightMode(mode);
        }
        return;
      }
    }
    if (currentMode !== null) {
      currentMode = null;
      resetHighlight();
    }
  }).on("mouseleave", resetHighlight);

  // Highlight function
  function highlightMode(mode) {
    chartG.selectAll("rect.cell")
      .attr("fill", d => d.mode===mode ? color(d.mode) : "#eee");

    pieGroup.selectAll("path")
      .attr("fill", d => d.data.mode===mode ? color(d.data.mode) : "#ddd");

    pieGroup.selectAll("text.stats").remove();
    const popRec = pctData.find(d=>d.mode===mode),
          emRec  = mappedData5.find(d=>d.mode===mode),
          emPct = (emRec ? emRec.emissions : 0) / totalEmissions * 100;
    if (popRec) {
      const txt = pieGroup.append("text").attr("class","stats")
        .attr("text-anchor","middle")
        .attr("y", pieRadius + 10)
        .attr("font-size",12);
      txt.append("tspan")
        .attr("x",0).attr("dy",0)
        .text(`${popRec.mode}: ${popRec.percent.toFixed(1)}% of population,`);
      txt.append("tspan")
        .attr("x",0).attr("dy","1.2em")
        .text(`accounting for ${emPct.toFixed(1)}% of tCO₂e`);
    }
  }

  // Reset function
  function resetHighlight() {
    chartG.selectAll("rect.cell")
      .attr("fill", d => color(d.mode));
    pieGroup.selectAll("path")
      .attr("fill", "#ddd");
    pieGroup.selectAll("text.stats").remove();
  }

  return svg.node();
}
```

## Further Discussion and Conclusion

There were many valuable insights from the 2024 Transportation Survey that would have not been arrived to easily without visualizing the data and putting the numbers into visualizations, minimizing the amount of work the human brain has to do to arrive at the insights the data is trying to relay. 

The questions were split into 4 main categories regarding transportation:
1. Mode
2. Time
3. Distance
4. Carbon Emissions

First we wanted to see what modes the community members take to campus, then what times do they get to campus, when do they depart, infering from that how many commnunity members are on campus at any time and at any location. We then looked at the departure choropleth map to see where they are departing from as a proxy to the distance that they travel to get to the different campuses. Finally to wrap up, we wanted to connect to the NYU 2040 carbon neutrality goal by calculating the amount of carbon emissions from from each mode of transportation based on the commute profile of each community member.

The findings underscore the critical role that transportation plays in the experiences of the faculty, students, and staff members, which revealed commute behaviors that are shaped by departure location, campus location, affiliation and the effect it has on the environment. 

Not surprising due to the setting being New York City, we observed a clear preference to publiuc transit, particularly the subway as the primary mode of transportation across all groups. However differences were noticed thereafter between professional staff, faculty and students. those were visualized via an interactive bar chart.

The time based analysis showed the fluctuations in arrival, departure and population on campus at any given time which were filtered by affiliation and location, as well as day of week. which could really inform potential needs for scheduling and infrastructure planning.

Spatial analysis through the choropleth map was created to show the geographical commuting patterns, which revealed the concenctrated clustered around campus locations, showing how students tend to reside closer to campus, meanwhile faculty and staff have a more distributed pattern. The spatial anaylsis also showed the different patterns, for different campuses.

The carbon emissions analysis connected the transportation survey with the 2040 Carbon Netutrality goal for NYU, by showing the scope 3 emissions from transportation. Some of the highlights there is the disproportionality between the carge carbon footprint of car commuting despite its low usage rate, which points towards a critical area for policy intervention.

The study offered NYU the ability to better understand its commuting patterns, gaining insights, to better work on policy, planning and enhance transportation while doing it sustainabily. Future research could build upon this initial data exploration, since this data really offeres an easy way to do an initial exploration, further questions will arrise and further analyses insue. That could be easily achieved by utilizing the CSV data and to take it a step further the visualizations could be utilized to compare the results to the 2019 transportation survey.

We hope that this work could drive further planning and policy to better serve the NYU community and lower our carbon footprint.

## Appendix

```js
import {checkbox} from "@observablehq/inputs";
```

```js
d3 = require('d3@7')
```

```js
Inputs = await import("@observablehq/inputs")
```

```js
rawLocations = Array.from(new Set(data2024_v21.map(d => d.primaryLocation)))
```

```js
locations = rawLocations.map(loc => loc === "" ? "None" : loc);
```

```js
fullWeightByZip = d3.rollup(
  data2024_v21,
  v => d3.sum(v, d => +d.weight),
  d => String(d.departureZipCode).padStart(5,"0")
);
```

```js
color = d3.scaleSequentialPow()
  .domain([0, maxWeightFull])
  .exponent(0.2)
  .interpolator(d3.interpolatePurples);
```

```js
maxWeightFull = d3.max(fullWeightByZip.values());

```

```js
features = radius_125.features.map(feat => {
  const raw = feat.properties.ZCTA5CE10 ?? feat.properties.ZIP ?? feat.id;
  feat.properties._zip5 = String(raw).padStart(5,"0");
  return feat;
});
```

```js
filteredData = {
  selectedLocations;
  selectedAffiliations;
  selectedModes;
  selectedModeFilter;

  return data2024_v21.filter(d => {
    // 1) normalize empty for loc/aff
    const loc = d.primaryLocation === "" ? "None" : d.primaryLocation;
    const aff = d.nyuAffiliations  === "" ? "None" : d.nyuAffiliations;

    // 2) location & affiliation
    if (
      !selectedLocations.includes(loc) ||
      !selectedAffiliations.includes(aff)
    ) return false;

    // 3) if no modes chosen → keep all
    if (selectedModes.length === 0) return true;

    // coerce to number ("" → 0, null/undefined → NaN)
    const val = key => +d[key];

    // (a) did they take any?
    const tookAny = selectedModes.some(key => val(key) > 0);

    if (selectedModeFilter === "Any of selected") {
      // OR‐logic: at least one
      if (!tookAny) return false;
    } else {
      // AND‐logic: Exact match → (took all selected) AND (took no others)
      const allModes = [
        "walkingTime_v2","subwayTime","railTime","busTime","shuttleTime",
        "pathTime","taxiTime","bikeTime","microTime","motorcycleTime",
        "carTime","carpoolTime","ferryTime","otherTime"
      ];

      // (b) did they take *every* selected?
      const tookAllSelected = selectedModes.every(key => val(key) > 0);

      // (c) did they take *no* unselected?
      const tookNoUnselected = allModes
        .filter(key => !selectedModes.includes(key))
        .every(key => val(key) <= 0 || isNaN(val(key)));

      if (!(tookAllSelected && tookNoUnselected)) return false;
    }

    return true;
  });
}
```

```js
affiliations = Array.from(new Set(data2024_v21.map(d => d.nyuAffiliations))).slice(0,3)
```

```js echo
all_modes = [
   'walkingTime_v2','subwayTime','railTime','busTime','shuttleTime',
   'pathTime','taxiTime','bikeTime','microTime','motorcycleTime',
   'carTime','carpoolTime','ferryTime','otherTime'];
```
