<script lang="ts">
	import { onMount } from 'svelte';
	import CharVideo from '$lib/3.mp4';

	let playing = false;
	let showVolume = false;
	let fullScreen = false;
	let currentTime = '0:00';
	let totalTime = '0:00';

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
		on:timeupdate={updateTime}
	>
		<source src={CharVideo} type="video/mp4" />
		<track kind="captions" />
	</video>
	<div class="absolute inset-0">
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
				{`${currentTime} / ${totalTime}`}
			</div>
		</div>
	</div>
</main>
