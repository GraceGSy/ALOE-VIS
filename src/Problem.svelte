<script>
	import * as d3 from "d3"

	export let processed_data = []
	export let students = []

	// $: console.log("processed", processed_data)

	$: if (processed_data) {

		let svg = d3.select("#temporal")

		let canvas = {"height": students.length * 30 + 75,
					  "width": window.innerWidth * 0.8,
					  "marginLeft": 220,
					  "marginTop": 50,
					  "marginRight": 100,
					  "marginBottom": 25}

		svg.attr("height", canvas.height)
			.attr("width", canvas.width)

		svg.on('click', function(e) {
			svg.transition()
				.duration(750)
				.call(zoom.transform, d3.zoomIdentity);

			e.stopPropagation()
		})

		d3.select("#clip-rect")
			.attr("x", canvas.marginLeft - 10)
			.attr("y", 0)
			.attr("height", canvas.height)
			.attr("width", canvas.width - canvas.marginLeft - canvas.marginRight + 20)

		const zoom = d3.zoom()
			.scaleExtent([1, 20])
			.on("zoom", zoomed);

		let yScale = d3.scaleBand()
			.domain(students)
			.range([canvas.marginTop, canvas.height-canvas.marginBottom]);

		// let xScale = d3.scaleTime()
		// 	.domain([d3.min(processed_data, d => d["normalizedStartTime"]), d3.max(processed_data, d => d["normalizedEndTime"])])
		// 	.range([canvas.margin, canvas.width - canvas.margin])

		let xScale = d3.scaleLinear()
			.domain([d3.min(processed_data, d => d["start"]), d3.max(processed_data, d => d["end"])])
			.range([canvas.marginLeft, canvas.width - canvas.marginRight])

		// console.log([d3.min(processed_data, d => d["start"]), d3.max(processed_data, d => d["end"])])

		// if (processed_data.length > 0) {
		// 			console.log(processed_data[5]["normalizedStartTime"], xScale(processed_data[5]["normalizedStartTime"]))

		// }

		let tooltip = d3.select("#tooltip")
					    .style("position", "absolute")
					    .style("visibility", "hidden")
					    .style("font-size", "8px")
					    .style("border", "solid 1px black")
					    .style("border-radius", "2.5px")
					    .style("padding", "5px")
					    .style("background", "white")
					    .text("");

		let vis_container = d3.select("#vis-container")

		// Add bar for each bin
		vis_container.selectAll(".bar")
			.data(processed_data)
		.join("rect")
			.attr("class", "bar")
			.attr("x", d => xScale(d["start"]))
			.attr("width", d => xScale(d['end']) - xScale(d['start']))
			.attr("y", d => yScale(d["userid"]) + 5)
			.attr("height", yScale.bandwidth() - 10)
			.attr("fill", d => {
				if (d["correctness"] == "CORRECT") {
					return "#c0d186"
				} else if (d["correctness"] == "INCORRECT") {
					return "#ed8b80"
				} else if (d["correctness"] == "HINT") {
					return "#ffd68a"
				} else {
					return "#bdbdbd"
				}
			})
			.attr("stroke", d => "black")
			.on("mouseover", function() {
				return tooltip.style("visibility", "visible"); })
			.on("mousemove", function(e, d) {
				let xPos = xScale(d.start + d.duration / 2)
				let yPos = yScale(d.userid) - yScale.bandwidth()
				let kc = d.kc_labels == "[]" ? "-" : d.kc_labels.split("'")[1]
				let selection = d.selection
				let input = d.input
				return tooltip.style("left", xPos.toString() + "px")
							  .style("top", yPos.toString() + "px")
							  .html("<strong>Knowledge Component:</strong> " + kc + "<br><strong>Selection:</strong> " + selection +"<br><strong>Input:</strong> " + input) ; })
			.on("mouseout", function() {
				return tooltip.style("visibility", "hidden"); });
			// .attr("stroke-width", d => d.selection == "done" ? "2px" : "1px")

		let start_only = processed_data.filter(d => d.action == "start new problem")

		// console.log(startOnly)

		let start_points = d3.select("#start-points")

		// Add start time indicator for each student
		start_points.selectAll(".start")
			.data(start_only)
		.join("circle")
			.attr("class", "start")
			.attr("cx", d => xScale(d["start"]))
			.attr("cy", d => yScale(d["userid"]) + yScale.bandwidth() / 2)
			.attr("r", 5)
			.attr("fill", "none")
			.attr("stroke", "black")
			.attr("stroke-width", "2px")

		// Add x axis
		// svg.append('g')
		// 	.attr("transform", `translate(0, ${canvas.height - canvas.margin})`)
		// 	.call(d3.axisBottom(xScale))

		svg.selectAll(".legendrect")
			.data(["incorrect", "correct", "hint"])
			.join("rect")
			.attr("class", "legendrect")
			.attr("x", canvas.width - canvas.marginRight + 12)
			.attr("y", (d, i) => canvas.marginTop + 15 * i)
			.attr("height", 10)
			.attr("width", 10)
			.attr("fill", d => {
				if (d == "correct") {
					return "#c0d186"
				} else if (d == "incorrect") {
					return "#ed8b80"
				} else if (d == "hint") {
					return "#ffd68a"
				} else {
					return "#bdbdbd"
				}
			})

		svg.selectAll(".legendtext")
			.data(["incorrect", "correct", "hint"])
			.join("text")
			.attr("class", "legendtext")
			.attr("x", canvas.width - canvas.marginRight + 25)
			.attr("y", (d, i) => canvas.marginTop + 15 * i + 7)
			.text(d => d)
			.attr("alignment-baseline", "middle")
			.attr("font-size", 8)

		let gy = svg.select('#y-axis');
		let yAxis = d3.axisLeft(yScale).tickSize(3).ticks(5).tickPadding([10])
        gy.transition()
        	.attr('transform', `translate(${canvas.marginLeft}, 0)`)
        	.call(yAxis)

        function formatTime(t) {
        	let seconds = t/1000

        	return seconds.toString() + 's'
        }

        let gx = svg.select('#x-axis');
        let xAxis = d3.axisBottom(xScale).tickSize(3).ticks(10).tickFormat(formatTime)
	    gx.transition()
	    	.attr('transform', `translate(0, ${canvas.height - canvas.marginBottom})`)
	    	.call(xAxis)

	    function zoomed({transform}) {
			console.log('zooming')
			xScale.range([canvas.marginLeft, canvas.width - canvas.marginRight].map(d => transform.applyX(d)));
			// const zx = transform.rescaleX(xScale).interpolate(d3.interpolateRound);
			svg.selectAll(".bar").attr("x", d => xScale(d["start"]))
								.attr("width", d => xScale(d['end']) - xScale(d['start']))
			svg.selectAll(".start").attr("cx", d => xScale(d["start"]))
			// vis_container.attr("transform", transform);
			gx.call(xAxis.scale(transform.rescaleX(xScale)))
			// gx.call(xAxis, zx);
			// gy.call(yAxis.scale(transform.rescaleX(yScale)));
			// gGrid.call(grid, zx, zy);
		}

		svg.call(zoom).call(zoom.transform, d3.zoomIdentity);

	}

</script>

<div>
	
</div>