<script lang="ts">
	import { onMount } from 'svelte';

	let video: HTMLVideoElement;
	let subtitle = '';
	onMount(() => {
		navigator.mediaDevices
			.getUserMedia({
				video: {
					width: 1280,
					height: 720
				}
			})
			.then((stream) => {
				const videoStream = new MediaStream(stream.getVideoTracks());
				video.srcObject = videoStream;
				video.onloadedmetadata = (e) => video.play();
			})
			.catch((e) => console.error(e.name + ': ' + e.message));

		const recognition = new window.webkitSpeechRecognition();
		recognition.lang = 'zh-cmn-Hans';
		recognition.continuous = true;
		recognition.interimResults = true;
		recognition.onresult = (e) => {
			const last = e.results[e.results.length - 1];
			// if (last.isFinal) {
			subtitle = last[0].transcript;
			// }
			// console.log(e.results);
		};
		recognition.onend = () => {
			// console.log('end');
		};
		recognition.onerror = (e) => {
			console.error(e);
		};
		recognition.start();
	});
</script>

<div class="video-container">
	<!-- svelte-ignore a11y-media-has-caption -->
	<video bind:this={video} />
	<p class="subtitle">{subtitle}</p>
</div>

<style>
	.video-container {
		width: 1280px;
		height: 720px;
		display: grid;
	}

	.video-container > * {
		grid-area: 1 / 1;
	}

	.video-container > .subtitle {
		font-family: 'Arial', 'Microsoft YaHei', '黑体', '宋体', sans-serif;
		font-size: 50px;
		font-weight: 700;
		-webkit-text-fill-color: white;
		-webkit-text-stroke: 2px black;

		padding: 0 2em;
		display: flex;
		align-items: end;
		justify-content: center;
		flex-wrap: wrap;
	}
</style>
