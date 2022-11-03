<script>
	import * as d3 from "d3"

	import Selector from './Selector.svelte'

	let promise = loadData()

	async function loadData() {
		const specs = await Promise.all([d3.csv('preliminary_tutor_transaction-raw.csv')])

		let data = specs[0]

		let data_by_problem = {}

		// let problems = new Set()

		for (let d of data) {
			// console.log(d)
			// console.log(d["start_state"], typeof d["start_state"])
			let start_state = d["start_state"].replaceAll("'", '"')
			let problem_content = JSON.parse(start_state)["initial_problem"]

			let student = d["userid"]

			if (!(problem_content in data_by_problem)) {
				data_by_problem[problem_content] = {}
			}

			let problem_data = data_by_problem[problem_content]

			if (!(student in problem_data)) {
				problem_data[student] = []
			}

			problem_data[student].push(d)
		}

		let data_by_student = {}
		let regex_by_student = {}

		let students = new Set()

		for (let d of data) {
			// console.log(d)
			// console.log(d["start_state"], typeof d["start_state"])
			let start_state = d["start_state"].replaceAll("'", '"')
			let problem_content = JSON.parse(start_state)["initial_problem"]

			// This line convert each step into a string for quick regex search
			// For now, this is only a simple record of correctness
			// Can be extended to include more information, such as timing data
			let problem_regex = d["action"] == "start new problem" ? "Snp" : d["correctness"][0]

			let student = d["userid"]

			if (!(student in data_by_student)) {
				data_by_student[student] = {}
				regex_by_student[student] = ''
			}

			regex_by_student[student] = regex_by_student[student] + problem_regex

			let student_data = data_by_student[student]

			if (!(problem_content in student_data)) {
				student_data[problem_content] = []
			}

			d['problem'] = problem_content

			student_data[problem_content].push(d)
		}

		// console.log(data_by_student)

		return [data_by_problem, data_by_student, regex_by_student]
	}

</script>

<div>
	{#await promise}
		<div />
	{:then specs}
		<Selector data={specs}/>
	{/await}
</div>