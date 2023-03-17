<script lang="ts">
	import { select_options, tick } from "svelte/internal";
	import type { CommonComponentParameters } from "../types/CommonComponentParameters";

	interface OptionVal {
		id: string;
		text: string;
		image: string;
	}

	export let params: CommonComponentParameters;
	export let schema: any;
	export let value: any;

	let match = '';
	let dropdownState: "open" | "closed" = "closed";
	let selected: OptionVal | undefined = undefined;
	let options = [] as OptionVal[];
	let url: string;
	let timerHandle: any = null;

	$: suggestions = schema['suggestions'] || '';
	$: readOnly = params.containerReadOnly || schema.readOnly || false;

	let input: HTMLInputElement;

	const searchRequest = (match: string) => {
		const items = suggestions;
		if (items.length > 0 && !(typeof items[0] === 'object')) {
			options = (items as string[]).map(s => ({ id: s, text: s, image: '' }));
		} else {
			options = items as OptionVal[];
		}
		options = options.filter(opt => opt.text.toLowerCase().includes(match.toLowerCase()))
		dropdownState = "open";
	}

	const toggleDropDown = async () => {
		dropdownState = dropdownState === "open" ? "closed" : "open";
	};

	const blur = async () => {
		console.log('blur');
		timerHandle = setTimeout(() => {
			console.log('closing');
			dropdownState = "closed";
			params.pathChanged(params.path, value);
		}, 300);
	}

	const handleSelect = (item: OptionVal) => () => {
		console.log('sel');
		if (timerHandle) {
			clearTimeout(timerHandle);
			timerHandle = null;
		}
		if (readOnly) return;
		value = item.id;
		params.pathChanged(params.path, item.id);
		dropdownState = "closed";
	};

	const keyup = (ev: KeyboardEvent) => {
		searchRequest((ev.currentTarget as HTMLInputElement)?.value);
	}

	$: selected = options.find(opt => opt.id === value);
</script>

<!-- event which calls pathChanged should be after all bindings so 'value' will have been updated -->
<svelte:component this={params.components['fieldWrapper']} {params} {schema}>
	<div class="sf-autocomplete" class:readonly={readOnly}>
		<div class="sf-selected-item input" on:click={toggleDropDown}>
			<input type="text" bind:value={value} on:keyup={keyup} bind:this={input} on:blur={blur}/>
		</div>

		<div style:display={dropdownState === "open" ? 'block' : 'none'} class="sf-items">
			{#each options as item (item.id)}
				<div on:click={handleSelect(item)} class:selected={value === item.text}>
					{#if item.image}
						<img src={item.image} alt={item.text}/>
					{/if}
					{item.text}
				</div>
			{/each}
		</div>
	</div>
</svelte:component>

<style>
	.sf-items div { cursor: pointer }
</style>