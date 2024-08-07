<script lang="ts">
	import { AppBar, ProgressBar } from "@skeletonlabs/skeleton"
	import { shuffleArray } from "$lib/utils/kana"
	import QuizInput from "./QuizInput.svelte"
	import QuizButton from "./QuizButton.svelte"
	import QuizResults from "./QuizResults.svelte"
	import { post } from "$lib/utils/api"

	interface KanaList {
		japanese: string
		romaji: string
	}

	export let data
	export let quiz: boolean
	export let hiragana: boolean
	export let groupName: string
	export let selectedGroup: Kana[]

	let currentJapaneseCharacter = ""
	let currentRomajiCharacter = ""
	let quizStage = 4
	let currentIndex = 0
	let correctKanaCount = 0
	let incorrectKanaCount = 0
	let shuffledKanaList: KanaList[] = []
	$: progressBarValue = (correctKanaCount / (shuffledKanaList.length * 3)) * 100

	const init = () => {
		const kanaList = selectedGroup.map((kana) => ({
			japanese: kana.japanese,
			romaji: kana.romaji
		}))

		shuffledKanaList = shuffleArray(kanaList)
	}

	const submitProgress = () => {
		const requestData = {
			userId: data.user.id,
			groupName,
			hiragana
		}
		post("/learn", requestData)
	}

	const saveUserProgress = () => {
		quizStage <= 3 ? correctKanaCount++ : submitProgress()
	}

	const handleRestudy = () => {
		quiz = false
	}

	init()
	submitProgress()
</script>

<AppBar class="mt-4 p-5 rounded-container-token" background="variant-ghost">
	Correctly answer each syllable 3 times in order to pass the lesson
</AppBar>
<div class="mt-4 flex flex-col bg-white/30 rounded-container-token dark:bg-black/30">
	<div class="flex justify-center">
		<div class="flex w-1/2 flex-col gap-4 p-20">
			{#if quizStage === 1}
				<QuizButton
					romajiToJapanese={false}
					bind:currentJapaneseCharacter
					{currentRomajiCharacter}
					{currentIndex}
					bind:incorrectKanaCount
					bind:quizStage
					{shuffledKanaList}
					{handleRestudy}
					{saveUserProgress}
				/>
			{:else if quizStage === 2}
				<QuizInput
					bind:currentJapaneseCharacter
					{currentRomajiCharacter}
					{currentIndex}
					bind:incorrectKanaCount
					bind:quizStage
					{shuffledKanaList}
					{handleRestudy}
					{saveUserProgress}
				/>
			{:else if quizStage === 3}
				<QuizButton
					romajiToJapanese={true}
					{currentJapaneseCharacter}
					bind:currentRomajiCharacter
					{currentIndex}
					bind:incorrectKanaCount
					bind:quizStage
					{shuffledKanaList}
					{handleRestudy}
					{saveUserProgress}
				/>
			{:else}
				<QuizResults {data} {hiragana} {incorrectKanaCount} {groupName} {handleRestudy} />
			{/if}
		</div>
	</div>
	<span class="mb-1 flex justify-center">
		{correctKanaCount}/{shuffledKanaList.length * 3}
	</span>
	<ProgressBar value={progressBarValue} max={100} />
</div>
