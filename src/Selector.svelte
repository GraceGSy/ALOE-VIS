<script>
	import ProblemWrapper from './ProblemWrapper.svelte'
	import StudentWrapper from './StudentWrapper.svelte'

	import Fa from 'svelte-fa'
  	import { faSquare } from '@fortawesome/free-solid-svg-icons'
	import { faAngleLeft } from '@fortawesome/free-solid-svg-icons'
  	import { faCircle } from '@fortawesome/free-regular-svg-icons'
  	// import { faMagnifyingGlass } from '@fortawesome/free-regular-svg-icons'

	export let data=[]

	let search_by = 'problem';

	let selected_problem;
	let selected_problem_data;
	let selected_student;
	let selected_student_data;

	let problems;
	let students;
	let problem_data;
	let student_data;

	let student_regex;
	let search_regex = [];
	let search_results = [];

	$: problems = Object.keys(data[0])
	$: students = Object.keys(data[1])
	$: problem_data = data[0]
	$: student_data = data[1]

	$: student_regex = data[2]
	// $: console.log(student_regex)

	$: if (selected_problem) {
		selected_problem_data = problem_data[selected_problem]
		// console.log(selected)
	}

	$: if (selected_student) {
		selected_student_data = student_data[selected_student]
		// console.log(selected)
	}

	// $: console.log(search_regex)

	function handleQuery(input) {
		if (input == "Back") {
			search_regex.pop()
			search_regex = [...search_regex]
		} else if (input == "Clear") {
			search_regex = []
			search_results = []
		} else {
			search_regex = [...search_regex, input]
		}
	}

	function handleSearch() {
		let students_found = []

		for (let s of students) {
			let s_regex = student_regex[s]
			let r = new RegExp(search_regex.join(''))

			if (s_regex.search(r) >= 0) {
				students_found.push(s)
			}
		}

		search_results = [...students_found]
	}

</script>

<style>
	#selectors {
		display: flex;
		align-items: flex-start;
	}

	#toggle {
		margin-right: 20px;
	}

	.dropdown {
		margin-bottom: 0px;
		height: 36px;
	}

	#search{
		display: flex;
		flex-direction: column;
		padding-left: 20px
	}

	#searchtools {
		display: flex;
		align-items: center;
	}

	#searchbar {
		display: flex;
		width: 100px;
		overflow: scroll;
		border: solid 1px gray;
		border-radius: 20px;
		align-items: center;
		padding-left: 15px;
		padding-right: 15px;
		margin-right: 10px;
		height: 36px;
	}

	#magnifyingglass {
		font-size: 18px;
		color: #cecece;
		margin: 1px;
	}

	.I {
		font-size: 18px;
		color: #ed8b80;
		margin: 1px;
	}

	.C {
		font-size: 18px;
		color: #c0d186;
		margin: 1px;
	}

	.H {
		font-size: 18px;
		color: #ffd68a;
		margin: 1px;
	}

	.Snp {
		font-size: 18px;
		margin: 1px;
	}

	button {
		margin: 0px;
		height: 36px;
	}
</style>

<div id="selectors">

	<div id="toggle">

		<label>
			<input type=radio bind:group={search_by} name="search_by" value="problem">
			Search by Problem
		</label>

		<label>
			<input type=radio bind:group={search_by} name="search_by" value="student">
			Search by Student
		</label>

	</div>

	{#if search_by == 'problem'}

		<select class="dropdown" bind:value={selected_problem}>
			{#each problems as p}
				<option value={p}>
					{p}
				</option>
			{/each}
		</select>

		<ProblemWrapper problem_data={selected_problem_data} />

	{:else}

		<select class="dropdown" bind:value={selected_student}>
			{#each students as s}
				<option value={s}>
					{s}
				</option>
			{/each}
		</select>

		<StudentWrapper student_data={selected_student_data} />

		<div id="search">

			<div id="searchtools">

				<div id="searchbar">

					<!-- <div id="magnifyingglass">
						<Fa icon={faMagnifyingGlass}/>
					</div> -->

					{#each search_regex as i}
						<div class={i}>
							<Fa icon={i == "Snp" ? faCircle : faSquare}/>
						</div>
					{/each}
				</div>

				<button on:click={() => handleQuery("I")}>
					<div class=I>
						<Fa icon={faSquare}/>
					</div>
				</button>

				<button on:click={() => handleQuery("C")}>
					<div class=C>
						<Fa icon={faSquare}/>
					</div>
				</button>

				<button on:click={() => handleQuery("H")}>
					<div class=H>
						<Fa icon={faSquare}/>
					</div>
				</button>

				<button on:click={() => handleQuery("Snp")}>
					<div class=Snp>
						<Fa icon={faCircle}/>
					</div>
				</button>

				<button on:click={() => handleQuery("Back")}>
					<div>
						<Fa icon={faAngleLeft}/> Back
					</div>
				</button>

				<button on:click={() => handleSearch()}>
					Search
				</button>

				<button on:click={() => handleQuery("Clear")}>
					Clear
				</button>

			</div>

			<div>
				{#each search_results as sr}
					<p>
						{sr}
					</p>
				{/each}
			</div>

		</div>

	{/if}
	
</div>