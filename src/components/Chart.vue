<template>
  <section class="chart">
    <section class="gantt">
      <svg></svg>
    </section>
  </section>
</template>

<script>
export default {
  name: 'Chart',
  data: () => ({
    width: window.innerWidth,
  }),
  mounted() {
    const self = this;
    const today = new Date();
    var mindate = this.$moment(today)._d,
        maxdate = this.$moment(mindate).add('14', 'days');

    const tasks = [
      {
        user_id: 1,
        user: 'user_1',
        isIncluded: true,
        text: 'name task 1',
        times: {
          starting_time: [2021,6,14],
          ending_time: [2021,6,20]
        }
      },
      {
        user_id: 2,
        user: 'user_2',
        isIncluded: true,
        text: 'name task 2',
        times: {
          starting_time: [2021,6,15],
          ending_time: [2021,6,16]
        }
      },
      {
        user_id: 2,
        user: 'user_2',
        isIncluded: true,
        text: 'name task 3',
        times: {
          starting_time: [2021,6,21],
          ending_time: [2021,6,23]
        }
      }
    ];

    const dataObj = [
      { id: 1, user: 'user 1'},
      { id: 2, user: 'user 2'},
      { id: 3, user: 'user 3'}
    ];

    var svg = this.$d3.select("svg"),
      margin = {
        top: 10,
        right: 10,
        bottom: 10,
        left: 10
      },
      width = self.width - margin.left - margin.right,
      height = dataObj.length * 100,
      g = svg.append("g").attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    var zoom = this.$d3.zoom()
      .scaleExtent([.5, 6])
      .on("zoom", zoomed)
        

    var x = this.$d3.scaleTime()
      .domain([mindate, maxdate])
      .range([250, width]);

    var y = this.$d3.scaleBand()
      .domain(dataObj.map(d => d.user).reverse())
      .rangeRound([height, 0])

    var xAxis = this.$d3.axisBottom(x)
      .ticks(self.$d3.time)
      .tickPadding(10)
      .tickSize(-height);

    var yAxis = this.$d3.axisRight(y)
      .ticks(dataObj.length)
      .tickSize(width)
      .tickPadding(-width)

    var gX = svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .attr("class", "axis axis--x")
      .style("font-size", "14px")
      .call(xAxis.tickFormat(self.$d3.timeFormat("%d %b")))

      // При развороте дат все крашется
      // .selectAll("text") 
      // .style("text-anchor", "end")
      // .attr("dx", "-.8em")
      // .attr("dy", ".15em")
      // .attr("transform", "rotate(-90)");

    var gY = svg.append("g")
      .attr("class", "axis axis--y")
      .style("font-size", "14px")
      .call(yAxis)
      .selectAll('text')
      .attr('fill', 'red')

    // BARS
    const bars = svg.selectAll(".bar")
        .data(tasks)
        .enter()
        .append('g')
    
    bars.append("rect")
        .attr("class", "bar")
        .attr("x", d => x(new Date(d.times.starting_time)))
        .attr("y", d => {
          const label = dataObj.find(obj => obj.id === d.user_id);
          const index = dataObj.indexOf(label)
          
          return index * 100
        })
        .attr("height", (height / dataObj.length) - 2)
        .attr("width", d => {
          return self.$d3.timeDay.count(new Date(d.times.starting_time), new Date(d.times.ending_time))  * (width - 250) / 14
        })
        .attr('fill', '#00ff00')

    svg.call(zoom);

    function zoomed() {
      var t = self.$d3.event.transform;
      var sx = t.rescaleX(x);

      gX.call(xAxis.scale(sx));
      bars.style('transform', `translateX(${t.x}px) scaleX(${t.k})`);
    }
  }
}
</script>

<style lang="scss">
.gantt {
  & svg {
    width: 100%;
    height: 900px;
  }
}
</style>
