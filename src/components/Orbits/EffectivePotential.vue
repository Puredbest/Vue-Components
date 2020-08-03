<template>
    <div>
        <div id="potentialPlot" style="height: 50vh;" >
        </div>
    </div>
</template>

<script>
import * as d3 from 'd3';

export default {
    name:'iv-eff-pot',
    props: {
    },
    mounted(){

        // let data = [[0, 0], [10, 10], [20, 40], [30, 90]];
        let data2 = [[90, 30], [40, 20], [10, 10], [0, 0]];

        let minX = 0;
        let maxX = 100;
        let minY = 0;
        let maxY = 100;
        let plotWidth = document.getElementById('potentialPlot').offsetWidth;
        let plotHeight = document.getElementById('potentialPlot').offsetHeight;
        let axWidth = plotWidth * 0.8;
        let axHeight = plotHeight * 0.8;

        var xScaler = d3.scaleLinear()
                         .domain([minX, maxX])
                         .range([0, axWidth]);

        var yScaler = d3.scaleLinear()
                        .domain([minY, maxY])
                        .range([axHeight, 0])

        var plotSvg = d3.select("#potentialPlot")
                        .append("svg")
                        .attr("width", plotWidth)
                        .attr("height", plotHeight)

        plotSvg.append("g")
               .attr("transform", "translate(50,10)")
               .attr("class", "yaxis")
               .call(d3.axisLeft().scale(yScaler));
        
        plotSvg.append("g")
                .attr("transform", "translate(50, " + axHeight+10 + ")")
                .attr("class", "xaxis")
                .call(d3.axisBottom().scale(xScaler));

        plotSvg.append("path")
               .data([data2])
               .attr("transform", "translate(50,10)")
               .attr("class", "line")
               .attr("stroke", "magenta")
               .attr("stroke-width", 2)
               .attr("fill", "none")
               .attr("d", d3.line()
                            .x(function(d) { return xScaler(d[0]); })
                            .y(function(d) { return yScaler(d[1]); })
                            .curve(d3.curveMonotoneX)
                    )

        function rescale(maxX, maxY) {
            xScaler.domain([0, maxX])
            yScaler.domain([0, maxY])
            plotSvg.select('.xaxis')
                   .transition()
                   .duration(1500)
                   .call(d3.axisBottom().scale(xScaler))
            plotSvg.select('.yaxis')
                   .transition()
                   .duration(1500)
                   .call(d3.axisLeft().scale(yScaler))
            plotSvg.select(".line")
                   .transition().duration(1500)
                   .attr("d", d3.line()
                            .x(function(d) { return xScaler(d[0]); })
                            .y(function(d) { return yScaler(d[1]); })
                            .curve(d3.curveMonotoneX)
                    )
        }
        
        setTimeout(rescale(90, 30), 2000);

    }
}
</script>

<style scoped>

</style>