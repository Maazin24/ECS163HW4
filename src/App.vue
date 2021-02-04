
<template>
  <div id="app">
    <YearlyOverviewScatterPlot
      class="scatter-plot"
      v-if="dataset"
      chartId="YearlyRepresentation"
      title="Yearly Flight Statistics"
      :dataset="dataset"
      attribX="Passengers"
      labelX="Passenger Count (in Millions)"
      attribY="Flights"
      labelY="Flight Count (in Thousands)"
      attribColor="Type_1"
      attribGlyph="AboveAverage"
      attribGlyphValNormal="False"
      attribGlyphValSpecial="True"
      @item-selected="YearSelected"
      :width="800"
      :height="500"
    />
    <YearDetailsBarChart
      class="bar-chart"
      v-if="datasetSelected"
      chartId="YearDetails"
      :title="`Passengers by Terminal in ${datasetSelected.name}`"
      :dataset="datasetSelected.entries" 
      attribX="name" 
      attribY="value"  
      labelY="Passenger Count (in Millions)"
      :width="800"
      :height="500"
      />
    <h1 v-else class="bar-chart-info">Select a Year from the Scatterplot</h1>
    <pie-chart :data="[['2006', 49],['2007',52],['2008',56],['2009',59],['2010',62]
    ,['2011',64],['2012',70],['2013',80],['2014',78],['2015',82],['2016',92],['2017',89],['2018',93],['2019',89]
    ]"></pie-chart>
  </div>
</template>

<script>
import * as d3 from "d3";

import YearlyOverviewScatterPlot from "./components/ScatterPlot.vue";
import YearDetailsBarChart from "./components/BarChart.vue";

export default {
  name: "App",
  components: {
    YearlyOverviewScatterPlot,
    YearDetailsBarChart,
  },
  data() {
    return {
      dataset: null,
      datasetSelected: null,
    };
  },
  created() {
    d3.csv("/data/LAXInfo.csv", d3.autoType).then((data) => {

      this.dataset = d3.map(data, (d) => {
        return Object.assign(d, {
          Passengers: d.PassCount,
          Flights: d.FlightCount,
        })
      })
    });
  },

      testdata2() {
    return {
      chartData: {
        '2017-05-13': 2,
        '2017-05-14': 5,
        '2017-05-15': 4,
        '2017-05-16': 10,
      }
    }
  },
  
  methods: {
    YearSelected(y) {
      if (y == null) {
        this.datasetSelected = null
        return
      }

      let YearData = d3.filter(this.dataset, d => d.Number == y.Number)[0]
      
      let entries = []
      for (const stat of ['Terminal1&2', 'Terminal3&4', 'Terminal5&6', 'Terminal7&8', 'InternationalTerminal', 'ImperialTerminal']) {
        entries.push({
          name: stat,
          value: YearData[stat]
        })
      }

      this.datasetSelected = {
        name: YearData.Name,
        entries: entries
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: hsl(210, 29%, 24%);
  display: grid;
  grid-template-rows: 5vh 60vh auto;
  grid-template-columns: 1fr 10fr 10fr 1fr;
  grid-template-areas: 
    ". . . ." 
    ". overview detail ."
    ". . . .";
}

.scatter-plot {
  grid-area: overview;
}

.bar-chart {
  grid-area: detail;
}

.bar-chart-info {
  grid-area: detail;
  color: rgb(214, 214, 214);
  margin: auto;
}
</style>

