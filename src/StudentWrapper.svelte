<script>
	import Student from './Student.svelte'

	export let student_data=[];

	let problems = [];
	let processed_data = [];

	$: problems = Object.keys(student_data)

	let d = JSON.parse(JSON.stringify(student_data));

	function processData(d) {
		let allData = [];

		for (let problem of problems) {
			let problemData = d[problem]

			// if (studentData.length <= 1) {
			// 	continue
			// }
			
			// allData = allData.concat(d[student].slice(0, studentData.length - 1))

			allData = allData.concat(problemData)

		}

		let previousEnd = 0

		for (let i = 0; i < allData.length; i++) {
			let step = allData[i]

			if (step.action == "start new problem") {
				if (i == 0) {
					// If this is the first step, go to next step
					step["duration"] = 0;
					step["start"] = 0;
					step["end"] = 0;
					continue;
				} else {
					// If this is not the first step, increment previousEnd
					let startTime = new Date(step["time"]);
					// let normalizedStartTime = previousEnd

					let lastStep = allData[i-1];
					let lastStepTime = new Date(lastStep["time"]);
					let duration = startTime - lastStepTime;

					previousEnd = previousEnd + duration;

					step["duration"] = 0;
					step["start"] = previousEnd;
					step["end"] = previousEnd;
					continue;
				}
			}

			let startTime = new Date(step["time"]);
			// let normalizedStartTime = previousEnd

			let lastStep = allData[i-1];
			let lastStepTime = new Date(lastStep["time"]);
			let duration = startTime - lastStepTime;
			step["duration"] = duration;
			step["start"] = previousEnd;
			step["end"] = previousEnd + duration;

			previousEnd = previousEnd + duration;
		}
		// console.log(allData)

		return allData
	}

	// $: console.log()

	$: processed_data = processData(JSON.parse(JSON.stringify(student_data)))

	// $: console.log(processed_data)

</script>

<div>
	<Student processed_data={processed_data} problems={problems}/>
</div>