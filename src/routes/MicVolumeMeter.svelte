<script lang="ts">
	import { onDestroy } from 'svelte';
	export let tracks: MediaStreamTrack[];

	let script: ScriptProcessorNode | null = null;
	let mic: MediaStreamAudioSourceNode | null = null;
	let volume = 0;

	$: if (tracks.length > 0) {
		const audioStream = new MediaStream(tracks);
		const context = new window.AudioContext();
		script = context.createScriptProcessor(2048, 1, 1);
		script.onaudioprocess = (e) => {
			const input = e.inputBuffer.getChannelData(0);
			var sum = 0.0;
			for (var i = 0; i < input.length; ++i) {
				sum += input[i] * input[i];
			}
			volume = Math.sqrt(sum / input.length);
		};
		const mic = context.createMediaStreamSource(audioStream);
		mic.connect(script);
		script.connect(context.destination);
	}

	onDestroy(() => {
		mic?.disconnect();
		script?.disconnect();
	});
</script>

<div
	class="bg-white h-2 w-48 basis-48 origin-left"
	style="transform: scaleX({volume.toFixed(2)});"
/>
