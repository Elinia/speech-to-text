<script lang="ts">
	import { onMount } from 'svelte';
	import MicToggle from './MicToggle.svelte';
	import MicVolumeMeter from './MicVolumeMeter.svelte';
	import SpeechToText from './SpeechToText.svelte';

	let video: HTMLVideoElement;

	let micEnabled = true;
	let audioTracks: MediaStreamTrack[] = [];
	onMount(() => {
		navigator.mediaDevices
			.getUserMedia({ video: true })
			.then((stream) => {
				const videoStream = new MediaStream(stream.getVideoTracks());
				video.srcObject = videoStream;
				video.onloadedmetadata = (e) => video.play();
			})
			.catch((e) => console.error(e.name + ': ' + e.message));
		navigator.mediaDevices
			.getUserMedia({ audio: true })
			.then((stream) => (audioTracks = stream.getAudioTracks()))
			.catch((e) => console.error(e.name + ': ' + e.message));
	});

	let speechToTextContent = '';
</script>

<div class="w-screen h-screen bg-black">
	<!-- svelte-ignore a11y-media-has-caption -->
	<video bind:this={video} class="absolute w-full top-16" />
	<div class="absolute top-4 left-4 h-12 flex items-center justify-items-start gap-1">
		<MicToggle tracks={audioTracks} on:enabled={(e) => (micEnabled = e.detail.enabled)} />
		<MicVolumeMeter tracks={audioTracks} />
	</div>
	<div class="absolute top-4 right-4 w-12 h-12">
		<SpeechToText enabled={micEnabled} on:text={(e) => (speechToTextContent = e.detail.content)} />
	</div>
	{#if micEnabled}
		<p class="text-content">{speechToTextContent}</p>
	{/if}
</div>

<style lang="postcss">
	video {
		max-height: calc(100% - 48px - 2rem);
	}

	.text-content {
		@apply w-full absolute bottom-0 px-4 mb-4 text-center;

		font-family: 'Arial', 'Microsoft YaHei', '黑体', '宋体', sans-serif;
		font-size: 2em;
		font-weight: 500;
	}
</style>
