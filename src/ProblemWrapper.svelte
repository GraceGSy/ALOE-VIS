<script>
	import Problem from './Problem.svelte'

	export let problem_data=[];

	let students = [];
	let processed_data = [];

	$: students = Object.keys(problem_data)

	let d = JSON.parse(JSON.stringify(problem_data));

	function processData(d) {
		for (let student of students) {
			let student_steps = d[student];

			// let previousEnd = new Date(new Date().setHours(0,0,0,0));
			let previousEnd = 0

			for (let i = 0; i < student_steps.length; i++) {
				let step = student_steps[i];

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

						let lastStep = student_steps[i-1];
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

				let lastStep = student_steps[i-1];
				let lastStepTime = new Date(lastStep["time"]);
				let duration = startTime - lastStepTime;
				step["duration"] = duration;
				step["start"] = previousEnd;
				step["end"] = previousEnd + duration;

				previousEnd = previousEnd + duration;

				// let nextStep = student_steps[i+1]
				// let nextStepTime = new Date(nextStep["time"])
				// let timeToNext = nextStepTime.getTime() - startTime.getTime()
				// let normalizedEndTime = new Date(normalizedStartTime.getTime() + timeToNext)

				// step["normalizedStartTime"] = normalizedStartTime
				// step["timeToNext"] = timeToNext
				// step["normalizedEndTime"] = normalizedEndTime

				// previousEnd = normalizedEndTime
			}
		// console.log(problem_data[student])
		}

		let allData = [];

		for (let student of students) {
			let studentData = d[student]

			// if (studentData.length <= 1) {
			// 	continue
			// }
			
			// allData = allData.concat(d[student].slice(0, studentData.length - 1))

			allData = allData.concat(d[student])

		}

		return allData
	}

	// $: console.log()

	$: processed_data = processData(JSON.parse(JSON.stringify(problem_data)))

	// $: console.log(processed_data)

</script>

<div>
	<Problem processed_data={processed_data} students={students}/>
</div>