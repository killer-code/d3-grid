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
    var mindate = new Date(2021,5,4),
        maxdate = new Date(2021,5,18);
    // 10,000 random data points
    var data = this.$d3.range(1, 10000).map(function(d) {
      return {
        i: d,
        x: mindate,
        y: 5,
      }
    });

    const tasks = [
      {
        user: 'user_1',
        isIncluded: true,
        text: 'name task 1',
        times: {
          starting_time: [2021,5,15],
          ending_time: [2021,5,20]
        }
      },
      {
        user: 'user_2',
        isIncluded: true,
        text: 'name task 2',
        times: {
          starting_time: new Date(2021,5,15),
          ending_time: new Date(2021,5,20)
        }
      },
      {
        user: 'user_2',
        isIncluded: true,
        text: 'name task 3',
        times: {
          starting_time: new Date(2021,5,25),
          ending_time: new Date(2021,6,20)
        }
      }
    ];

    const dataObj = [
      {
        id: 1,
        user: 'user 1',
        start: new Date(),
        end: new Date(2021,5,10)
      },
      {
        id: 2,
        user: 'user 2',
        start: new Date(),
        end: new Date(2021,5,10)
      }
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

    // large "endless" zoom
    var zoom = this.$d3.zoom()
      .scaleExtent([1, 6])
      .on("zoom", zoomed);

    var x = this.$d3.scaleTime()
      .domain([mindate, maxdate])
      .range([250, width]);

    var y = this.$d3.scaleBand()
      .domain(dataObj.map(d => d.user))
      .rangeRound([height, 0])

    var xAxis = this.$d3.axisBottom(x)
      .ticks((width - 250) / 100)
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
      .call(yAxis);

    // BARS
    const bars = svg.selectAll(".bar")
        .data(tasks)
        .enter()
        .append('g')
    
    bars.append("rect")
        .attr("class", "bar")
        .attr("x", d => 250 )
        .attr("y", d => 100)
        .attr("height", height / dataObj.length)
        .attr("width", d => {
          let days = self.$moment.duration(self.$moment(tasks[0].times.ending_time).diff(tasks[0].times.starting_time, 'days'))
          days = Number(days);
          return days * (width - 250) / 14
        });

    svg.call(zoom);
    console.log((window.innerWidth - 250) / 14)

    function zoomed() {
      var t = self.$d3.event.transform,
      sx = t.rescaleX(x);

      gX.call(xAxis.scale(sx));

      // updateData(sx)
    }

    // classic enter, update, exit pattern
    // function updateData(sx, sy) {

    //   // filter are data to those points in range
    //   var f = data.filter(function(d) {
    //     return (
    //       d.x > sx.domain()[0] &&
    //       d.x < sx.domain()[1] &&
    //       d.y > sy.domain()[0] &&
    //       d.y < sy.domain()[1]
    //     )
    //   });

    //   var s = g.selectAll(".point")
    //     .data(f, function(d) {
    //       return d.i;
    //     });

    //   // remove those out of range
    //   s.exit().remove();

    //   // add the new ones in range
    //   s = s.enter()
    //     .append('circle')
    //     .attr('class', 'point')
    //     .attr('r', 10)
    //     .style('fill', 'steelblue')
    //     .merge(s);

    //   // update all in range
    //   s.attr('cx', function(d) {
    //       return sx(d.x);
    //     })
    //     .attr('cy', function(d) {
    //       return sy(d.y);
    //     });
    // }
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
