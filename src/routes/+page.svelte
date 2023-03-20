<script lang="ts">
	import { onMount } from 'svelte';
	import CGIShortFilm from '$lib/CGI.mp4';
	import char1 from '$lib/1.png';
	import char2 from '$lib/2.png';
	import char3 from '$lib/3.png';
	import char4 from '$lib/4.png';
	import characters_timestamps from '$lib/characters_timestamps.json';
	const characterImages = [char1, char2, char3, char4];

	let playing = false;
	let showVolume = false;
	let fullScreen = false;
	let xRay = false;
	let individual = false;
	let currentTime: string;
	let totalTime: string;
	let characters: {
		index: number;
		timestamps: number[][];
		title: string;
		description: string;
	}[];

	function updateTime() {
		const video = document.getElementById('video') as HTMLVideoElement;
		const videoRange = document.getElementById('videoRange') as HTMLInputElement;
		videoRange.value = video.currentTime * (100 / video.duration) + '';
		let currentMinutes = Math.floor(video.currentTime / 60) as any;
		let currentSeconds = Math.floor(video.currentTime - currentMinutes * 60) as any;
		let totalMinutes = Math.floor(video.duration / 60) as any;
		let totalSeconds = Math.floor(video.duration - totalMinutes * 60) as any;
		if (currentMinutes < 10) currentMinutes = '0' + currentMinutes;
		if (currentSeconds < 10) currentSeconds = '0' + currentSeconds;
		if (totalMinutes < 10) totalMinutes = '0' + totalMinutes;
		if (totalSeconds < 10) totalSeconds = '0' + totalSeconds;

		currentTime = currentMinutes + ':' + currentSeconds;
		totalTime = totalMinutes + ':' + totalSeconds;
	}
	function reset() {
		const video = document.getElementById('video') as HTMLVideoElement;
		setTimeout(() => {
			if (video.paused && playing) {
				video.play();
			}
		}, 10);
	}
	function updateVolume() {
		const video = document.getElementById('video') as HTMLVideoElement;
		const volumeRange = document.getElementById('volumeRange') as any;
		video.volume = volumeRange.value / 100;
		reset();
	}
	function updateVideoRange() {
		const video = document.getElementById('video') as HTMLVideoElement;
		const videoRange = document.getElementById('videoRange') as HTMLInputElement;
		video.currentTime = video.duration * (+videoRange.value / 100);
		reset();
	}
	function forward() {
		const video = document.getElementById('video') as HTMLVideoElement;
		const videoRange = document.getElementById('videoRange') as HTMLInputElement;
		video.currentTime += 5;
		videoRange.value = (video.currentTime / video.duration) * 100 + '';
		reset();
	}
	function backward() {
		const video = document.getElementById('video') as HTMLVideoElement;
		const videoRange = document.getElementById('videoRange') as HTMLInputElement;
		video.currentTime -= 5;
		videoRange.value = (video.currentTime / video.duration) * 100 + '';
		reset();
	}
	function updateCharacters() {
		const video = document.getElementById('video') as HTMLVideoElement;
		const time = video.currentTime;
		characters = characters_timestamps.filter((char) => {
			let present = false;
			for (const timestamp in char.timestamps) {
				if (time >= +char.timestamps[timestamp][0] && time <= +char.timestamps[timestamp][1]) {
					present = true;
					break;
				}
			}
			return present;
		});
	}
	function togglePlay() {
		playing = !playing;
		const video = document.getElementById('video') as HTMLVideoElement;
		if (playing) video.play();
		else video.pause();
	}
	function toggleFullScreen() {
		if (!fullScreen) document.documentElement.requestFullscreen();
		else document.exitFullscreen();
		fullScreen = !fullScreen;
	}
	onMount(() => {
		setTimeout(() => {
			updateTime();
			const video = document.getElementById('video') as HTMLVideoElement;
			let playPromise = video.play();
			if (playPromise !== undefined) {
				playPromise
					.then((_) => {
						playing = true;
					})
					.catch((error) => {
						playing = false;
						console.log('Error in autoplay');
					});
			}
		}, 500);
	});
</script>

<main class="w-screen h-screen bg-black relative overflow-hidden text-white">
	<video
		class="absolute w-full h-[80%] object-cover top-1/3 -translate-y-1/3"
		id="video"
		on:timeupdate={() => {
			updateTime();
			updateCharacters();
		}}
	>
		<source src={CGIShortFilm} type="video/mp4" />
		<track kind="captions" />
	</video>
	<div class="absolute inset-0">
		<div class="absolute top-5 left-10 text-lg">
			<button
				on:click={() => {
					xRay = !xRay;
				}}
				>X-Ray {#if xRay}
					<i class="fas fa-angle-up" />
				{:else}
					<i class="fas fa-angle-down" />
				{/if}</button
			>
		</div>
		<div class="absolute top-5 right-32 text-lg">
			<button
				on:click={() => {
					individual = !individual;
				}}
				>Individual {#if individual}
					<i class="fas fa-angle-up" />
				{:else}
					<i class="fas fa-angle-down" />
				{/if}</button
			>
		</div>
		<div class="absolute top-5 right-10 flex gap-5">
			<button
				class="text-white text-opacity-80 hover:text-opacity-100"
				on:click={() => {
					showVolume = !showVolume;
				}}
			>
				<i class="fas fa-volume-up fa-lg" />
			</button>
			<button class="text-white text-opacity-80 hover:text-opacity-100" on:click={toggleFullScreen}>
				{#if fullScreen}
					<i class="fas fa-compress-alt" />
				{:else}
					<i class="fas fa-expand-alt fa-lg" />
				{/if}
			</button>
		</div>
		{#if showVolume}
			<div class="absolute top-[103px] right-[30px]">
				<input
					type="range"
					class="outline-none rounded-xl cursor-pointer bg-gray-500 -rotate-90 h-1"
					id="volumeRange"
					on:change={updateVolume}
				/>
			</div>
		{/if}
		<div class="absolute bottom-14 flex w-full justify-center gap-5">
			<button
				class="text-white text-opacity-80 hover:text-opacity-100 transition-all duration-150"
				on:click={backward}
			>
				<i class="fas fa-backward fa-lg" />
			</button>
			<button
				class="text-white text-opacity-80 hover:text-opacity-100 transition-all duration-150"
				on:click={togglePlay}
			>
				{#if playing}
					<i class="fas fa-pause fa-lg" />
				{:else}
					<i class="fas fa-play fa-lg" />
				{/if}
			</button>
			<button
				class="text-white text-opacity-80 hover:text-opacity-100 transition-all duration-150"
				on:click={forward}
			>
				<i class="fas fa-forward fa-lg" />
			</button>
		</div>
		<div class="absolute bottom-0 w-full px-5">
			<div>
				<input
					type="range"
					class="w-full outline-none rounded-xl cursor-pointer bg-gray-500 h-1"
					value="0"
					id="videoRange"
					on:change={updateVideoRange}
				/>
			</div>
			<div>
				{currentTime && totalTime ? `${currentTime} / ${totalTime}` : '0:00 / 0:00'}
			</div>
		</div>
	</div>
	{#if xRay && characters}
		<div class="absolute top-12 left-10 w-1/3 flex flex-col gap-2">
			{#each characters as character}
				<div class="w-full h-32 rounded bg-black bg-opacity-40 flex">
					<div class="w-1/4">
						<img
							class="w-full h-full object-cover"
							src={characterImages[character.index - 1]}
							alt={character.title}
						/>
					</div>
					<div class="w-3/4 flex flex-col justify-center pl-2">
						<h1 class=" font-bold">{character.title}</h1>
						<p>{character.description}</p>
					</div>
				</div>
			{/each}
		</div>
	{/if}
	{#if individual}
		<div class="absolute top-12 right-32 w-1/3 flex flex-col gap-2">
			{#each characters_timestamps as character}
				<a class="w-full h-32 rounded bg-black bg-opacity-40 flex" href="/{character.index}">
					<div class="w-1/4">
						<img
							class="w-full h-full object-cover"
							src={characterImages[character.index - 1]}
							alt={character.title}
						/>
					</div>
					<div class="w-3/4 flex flex-col justify-center pl-2">
						<h1 class=" font-bold">{character.title}</h1>
						<p>{character.description}</p>
					</div>
				</a>
			{/each}
		</div>
	{/if}
</main>
