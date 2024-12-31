<script lang="ts">
	import { getContext } from 'svelte';
	// import CitationsModal from './CitationsModal.svelte';
	import Collapsible from '$lib/components/common/Collapsible.svelte';
	import ChevronDown from '$lib/components/icons/ChevronDown.svelte';
	import ChevronUp from '$lib/components/icons/ChevronUp.svelte';

	const i18n = getContext('i18n');

	export let toolCalls = [];

	let showCitationModal = false;
	let selectedCitation: any = null;
	let isCollapsibleOpen = false;


	$: {
		toolCalls = toolCalls.reduce((acc, toolCall) => {
			if (Object.keys(toolCall).length === 0) {
				return acc;
			}
			console.log('toolCall', toolCall);
			acc.push(toolCall);
			return acc;
		}, []);
	}
</script>


{#if toolCalls.length > 0}
	<div class=" py-0.5 -mx-0.5 w-full flex gap-1 items-center flex-wrap">
		<div class="flex text-xs font-medium">
			{#each toolCalls as toolCall, idx}
				<div
					class="flex-1 mx-1 line-clamp-1 text-black/60 hover:text-black dark:text-white/60 dark:hover:text-white transition"
				>
					{toolCall.name}
				</div>
			{/each}
		</div>
	</div>
{/if}
