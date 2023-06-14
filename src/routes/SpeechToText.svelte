<script lang="ts">
	import Menu, { SelectionGroup, SelectionGroupIcon } from '@smui/menu';
	import { Item, Meta, Label, Separator, Text } from '@smui/list';
	import Checkbox from '@smui/checkbox';
	import SettingsBtn from './SettingsBtn.svelte';
	import { createEventDispatcher } from 'svelte';

	export let enabled: boolean;

	type Language = 'zh-cmn-Hans' | 'en-US';
	const SUPPORTED_LANG: Language[] = ['zh-cmn-Hans', 'en-US'];
	const SUPPORTED_LANG_TEXT: Record<Language, string> = {
		'zh-cmn-Hans': '简体中文',
		'en-US': 'English'
	};
	let menu: Menu;

	let speechToTextContent = '';
	let recognition: null | SpeechRecognition = null;

	function stopSpeechRecognition() {
		if (recognition) {
			recognition.abort();
			recognition.stop();
			speechToTextContent = '';
		}
	}

	function startSpeechRecognition(lang: Language) {
		stopSpeechRecognition();
		recognition = new webkitSpeechRecognition();
		recognition.lang = lang;
		recognition.continuous = true;
		recognition.interimResults = true;
		recognition.onresult = (e) => {
			// console.log(e);
			const last = e.results[e.results.length - 1];
			speechToTextContent = last[0].transcript;
		};
		recognition.onend = () => {
			// console.log('end');
		};
		recognition.onerror = (e) => {
			if (e.error === 'no-speech') {
				startSpeechRecognition(lang);
			} else {
				console.error(e.error);
			}
		};
		recognition.start();
	}

	let speechToTextEnabled = true;
	let currentLang: Language = 'zh-cmn-Hans';
	$: if (enabled && speechToTextEnabled) {
		startSpeechRecognition(currentLang);
	} else {
		stopSpeechRecognition();
	}

	const dispatch = createEventDispatcher();
	$: dispatch('text', { content: speechToTextContent });
</script>

<div class="w-12 h-12">
	<SettingsBtn on:click={() => menu.setOpen(true)} />
	<Menu bind:this={menu} class="top-1 right-1">
		<div class="mdc-deprecated-list">
			<Item on:SMUI:action={() => (speechToTextEnabled = !speechToTextEnabled)}>
				<Label>Speech To Text</Label>
				<Meta>
					<Checkbox checked={speechToTextEnabled} />
				</Meta>
			</Item>
			<Separator />
			<SelectionGroup>
				{#each SUPPORTED_LANG as item}
					<Item on:SMUI:action={() => (currentLang = item)} selected={currentLang === item}>
						<SelectionGroupIcon>
							<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"
								><title>check</title><path
									d="M21,7L9,19L3.5,13.5L4.91,12.09L9,16.17L19.59,5.59L21,7Z"
								/></svg
							>
						</SelectionGroupIcon>
						<Text>{SUPPORTED_LANG_TEXT[item]}</Text>
					</Item>
				{/each}
			</SelectionGroup>
		</div>
	</Menu>
</div>
