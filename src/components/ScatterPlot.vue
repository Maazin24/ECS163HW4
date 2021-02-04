<!-- Your HTML goes here -->
<template>
  <div>
    <h1>{{title}}</h1>
    <svg :class="`scatterplot-svg-${chartId}`" :viewBox="viewBox">

        <!-- Chart and Axes -->
        <g :transform="`translate(${margin.left}, ${margin.top})`">
            <g class="plot"></g>
            <g class="x-axis" :transform="`translate(0, ${height})`"></g>
            <g class="y-axis"></g>
        </g>

        <!-- Axis Labels -->
        <g :transform="`translate(${margin.left + (width/2.5)}, ${margin.top + margin.bottom/1.5 + height})`">
            <text>{{labelX}}</text>
        </g>
        <g :transform="`translate(${margin.left/4}, ${margin.top + (height/1.8)}) rotate(-90)`">
            <text>{{labelY}}</text>
        </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "ScatterPlotOfYearlyData",
  props: {
      chartId: {
          required: true,
          type: String
      },
      title: {
          type: String
      },
    dataset: {
      required: true,
      type: Array,
    },
    attribX: {
      required: true,
      type: String,
    },
    labelX: {
        type: String
    },
    attribY: {
      required: true,
      type: String,
    },
    labelY: {
        type: String
    },
    attribColor: {
      required: false,
      type: String,
    },
    attribGlyph: {
      reqired: true,
      type: String,
    },
    attribGlyphValNormal: {
        required: true,
    },
    attribGlyphValSpecial: {
        required: true,
    },
    height: {
      required: true,
      type: Number,
    },
    width: {
      required: true,
      type: Number,
    },
  },
  data() {
    return {
      x: null,
      y: null,
      color: null,
      margin: {
          left: 75,
          top: 50,
          right: 20,
          bottom: 75,
      }
    };
  },
  mounted() {
    this.init();
    this.generateArc();
  },
  methods: {
    init() {

      this.x = d3.scaleLinear();
      this.y = d3.scaleLinear();
      this.color = d3.scaleOrdinal();

      this.update();
    },
    update() {
      this.x.domain(d3.extent(this.dataset, d => d[this.attribX])).range([0, this.width])
      this.y.domain(d3.extent(this.dataset, d => d[this.attribY])).range([this.height, 0])
      this.color.domain(this.dataset, d => d[this.attribColor]).range(d3.schemeTableau10)

      this.renderScatter();
      this.renderXAxis();
      this.renderYAxis();
    },
    renderScatter() {
      // Plot normal points
      d3.select(`.scatterplot-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("circle")
        .data(this.dataGlyphNormal)
        .join("circle")
        .transition()
        .duration(200)
        .attr("cx", (d) => this.x(d[this.attribX]))
        .attr("cy", (d) => this.y(d[this.attribY]))
        .attr("fill", (d) => this.color(d[this.attribColor]))

      d3.select(`.scatterplot-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("rect")
        .data(this.dataGlyphSpecial)
        .join("rect")
        .transition()
        .duration(200)
        .attr("x", (d) => this.x(d[this.attribX]) - 5)
        .attr("y", (d) => this.y(d[this.attribY]) - 5)
        .attr("height", 10)
        .attr("width", 10)
        .attr("fill", (d) => this.color(d[this.attribColor]))

      d3.select(`.scatterplot-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("rect")
        .on('click', (event, d) => {
            this.itemSelected(d)
        })        
        .on('mouseover', function() {
            d3.select(this).attr('class', 'hover').attr('height', 15).attr('width', 15)
        })
        .on('mouseout', function() {
            d3.select(this).attr('class', null).attr('height', 10).attr('width', 10)
        })

      d3.select(`.scatterplot-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("circle")
        .on('click', (event, d) => {
            this.itemSelected(d)
        })
        .on('mouseover', function() {
            d3.select(this).attr('class', 'hover')
        })
        .on('mouseout', function() {
            d3.select(this).attr('class', null)
        })
    },
    renderXAxis() {
        let xAxis = d3.axisBottom(this.x)

        d3.select(`.scatterplot-svg-${this.chartId}`)
          .select('.x-axis')
          .call(xAxis)
    },
    renderYAxis() {
        let yAxis = d3.axisLeft(this.y)

        d3.select(`.scatterplot-svg-${this.chartId}`)
          .select('.y-axis')
          .call(yAxis)
    },
    itemSelected(d) {
        console.log(`Selected ${d}`)
        this.$emit('item-selected', d)
    }
  },
  computed: {
    dataGlyphSpecial() {
      return d3.filter(this.dataset, (d) => d[this.attribGlyph] == this.attribGlyphValSpecial);
    },
    dataGlyphNormal() {
      return d3.filter(this.dataset, (d) => d[this.attribGlyph] == this.attribGlyphValNormal);
    },
    viewBox() {
      return `0 0 ${this.width+this.margin.left+this.margin.right} ${this.height+this.margin.top+this.margin.bottom}`
    }
  },
};
</script>

<style>
.plot rect {
    fill-opacity: 0.5;
}

.plot rect.hover {
    fill-opacity: 1;
    stroke: black;
}

.plot circle {
    fill-opacity: 0.5;
    r: 5;
}

.plot circle.hover {
    fill-opacity: 1;
    r: 7.5;
    stroke: black;
}
</style>