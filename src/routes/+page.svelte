<script>
	const rawQuestions = [
		{
			text: 'Eg veit at du er lei av meg no, Kine, men kva er kløyvd infinitiv?',
			options: [
				'Spør du meg (Albert), har eg framleis inga aning.',
				'https://sprakradet.no/spraksporsmal-og-svar/kloyvd-infinitiv-/',
				'Det ligg i ordet: oppdelt infinitt verbalform',
				'Når eit verb kan ha to ulike bøyingsformer i fortid.'
			],
			correct: 0,
			explanation:
				'Eg har faktisk ikkje snøring, men du kan jo prøve å tyde det som står på Språkrådet.'
		}
	];

	function shuffleArray(array) {
		const shuffled = [...array];
		for (let i = shuffled.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
		}
		return shuffled;
	}

	function prepareQuestions() {
		return rawQuestions.map((question) => {
			const optionsWithIndices = question.options.map((option, index) => ({
				text: option,
				originalIndex: index
			}));

			const shuffledOptions = shuffleArray(optionsWithIndices);

			const newCorrectIndex = shuffledOptions.findIndex(
				(option) => option.originalIndex === question.correct
			);

			return {
				...question,
				options: shuffledOptions.map((option) => option.text),
				correct: newCorrectIndex
			};
		});
	}

	let screen = $state('start');
	let questions = $state([]);
	let currentQuestionIndex = $state(0);
	let score = $state(0);
	let answered = $state(false);
	let chosenAnswer = $state(null);

	const currentQuestion = $derived(questions[currentQuestionIndex]);
	const resultMessage = $derived.by(() => {
		const percentage = Math.round((score / questions.length) * 100);

		if (percentage >= 80) return 'Det er bekymringsverdig kor god du er på nynorsk.';
		if (percentage >= 50)
			return 'Bra jobba! No står det berre att å spørja seg sjølv kvifor ein veit så mykje om nynorsk.';
		return 'Du ville ha fått ein høgare poengsum ved å gjetta heilt tilfeldig';
	});

	function startQuiz() {
		if (questions.length === 0) {
			questions = prepareQuestions();
		}
		currentQuestionIndex = 0;
		score = 0;
		showQuestion();
		screen = 'quiz';
	}

	function showQuestion() {
		answered = false;
		chosenAnswer = null;
	}

	function selectOption(index) {
		if (answered) return;

		answered = true;
		chosenAnswer = index;

		if (index === currentQuestion.correct) {
			score++;
		}
	}

	function nextQuestion() {
		currentQuestionIndex++;
		if (currentQuestionIndex < questions.length) {
			showQuestion();
		} else {
			showResults();
		}
	}

	function showResults() {
		screen = 'results';
	}

	function optionClasses(index) {
		const base = 'flex items-center rounded-xl border-2 text-sm md:text-base';

		if (!answered) {
			return `${base} border-neutral-300 bg-neutral-200 hover:border-neutral-400 hover:bg-neutral-300`;
		}

		if (index === currentQuestion.correct) {
			return `${base} border-emerald-600 bg-emerald-600/10`;
		}

		if (index === chosenAnswer) {
			return `${base} border-orange-600 bg-orange-600/10`;
		}

		return `${base} border-neutral-300 bg-neutral-200 opacity-70`;
	}

	function explanationClasses() {
		const base =
			'flex min-h-24 w-full max-w-[600px] items-center justify-center self-center rounded-md border-l-4 p-4 text-center text-lg break-words text-balance';

		if (!answered) {
			return `${base} invisible`;
		}

		return chosenAnswer === currentQuestion.correct
			? `${base} border-emerald-600 bg-emerald-600/10`
			: `${base} border-orange-600 bg-orange-600/10`;
	}
</script>

<svelte:head>
	<title>Nynorskquizen</title>
	<link rel="icon" href="https://fav.farm/🇳🇴" />
</svelte:head>

<div
	class="flex min-h-dvh justify-center bg-white p-2 font-sans text-neutral-900 md:p-5"
	aria-live="polite"
>
	<div class="flex w-full max-w-[800px] flex-col">
		{#if screen === 'start'}
			<section
				class="flex flex-1 flex-col items-center justify-between px-3 py-8 text-center md:px-5 md:py-12"
			>
				<div class="flex flex-1 flex-col items-center justify-center gap-12 md:gap-16">
					<header class="flex flex-col items-center gap-5 md:gap-7">
						<h1 class="font-mono text-[2.5rem] font-bold text-amber-600 md:text-[4rem]">
							Nynorsk er enkelt
						</h1>
						<p class="text-lg text-neutral-600 italic md:text-2xl">
							Nynorsk er eit skriftspråk med standardiserte reglar og einsarta grammatikk.
						</p>
					</header>
					<button
						class="w-full max-w-[200px] rounded-xl bg-amber-600 px-[30px] py-4 text-base font-bold text-neutral-900 hover:bg-amber-700 md:max-w-[250px] md:text-lg"
						onclick={startQuiz}
					>
						Byrje quizen
					</button>
				</div>

				<footer
					class="mt-auto flex flex-col items-center justify-center gap-2 pt-4 pb-3 text-sm text-neutral-600"
				>
					<p>manifestert med blod, sveitte og tårer av albert</p>
				</footer>
			</section>
		{:else if screen === 'quiz' && currentQuestion}
			<section class="flex flex-1 flex-col gap-[15px] p-3 md:p-5">
				<div
					class="flex h-10 w-full items-center justify-center text-sm text-neutral-600 md:text-base"
				>
					Spørsmål {currentQuestionIndex + 1} av {questions.length}
				</div>

				<div
					class="flex min-h-16 w-full max-w-[600px] items-center justify-center self-center rounded-xl border-2 border-neutral-300 bg-neutral-100 p-4 text-center font-mono text-base text-balance break-words text-neutral-900 shadow-inner md:p-5 md:text-[1.25rem]"
				>
					<div>{currentQuestion.text}</div>
				</div>

				<div class={explanationClasses()}>
					{#if answered}
						<p>{@html currentQuestion.explanation}</p>
					{/if}
				</div>

				<ul class="mt-auto flex w-full max-w-[600px] flex-col gap-3 self-center md:gap-[15px]">
					{#each currentQuestion.options as option, index}
						<li class={optionClasses(index)}>
							<button
								class="flex w-full items-center gap-[15px] rounded-[inherit] bg-transparent px-[15px] py-3 text-left font-mono text-neutral-900 md:px-5 md:py-[15px]"
								onclick={() => selectOption(index)}
							>
								<span
									class="flex h-7 w-7 shrink-0 items-center justify-center rounded-lg bg-neutral-300 font-sans text-sm font-bold md:h-8 md:w-8 md:text-base"
								>
									{index + 1}
								</span>
								<span class="flex-1 font-bold break-words text-neutral-900">{option}</span>
							</button>
						</li>
					{/each}
				</ul>

				<button
					class="w-full max-w-[200px] self-center rounded-xl bg-amber-600 px-[30px] py-[15px] text-base font-bold text-neutral-900 hover:bg-amber-700 md:max-w-[250px] md:text-lg"
					class:invisible={!answered}
					onclick={nextQuestion}
				>
					Neste spørsmål
				</button>
			</section>
		{:else if screen === 'results'}
			<section class="flex flex-1 flex-col px-3 py-8 text-center md:px-5 md:py-12">
				<div class="flex flex-1 flex-col items-center justify-center gap-7">
					<h2 class="text-[2rem] text-amber-600 md:text-[3rem]">Du overlevde!</h2>
					<div class="text-[3rem] font-bold text-amber-600 md:text-[5rem]">
						{score} / {questions.length}
					</div>
					<p class="max-w-[32rem] text-lg text-balance text-neutral-700">{resultMessage}</p>
				</div>

				<footer
					class="flex flex-col items-center justify-center gap-2 pb-3 text-sm text-neutral-600"
				>
					<p>manifestert med blod, sveitte og tårer av albert</p>
				</footer>
			</section>
		{/if}
	</div>
</div>
