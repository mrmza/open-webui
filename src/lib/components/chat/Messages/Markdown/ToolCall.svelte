
<script lang="ts">

	import Plus from '$lib/components/icons/Check.svelte';


	export let token;
	export let toolCalls = [];
	export let onClick: Function = () => {};

	let id = '';
	let isLoading = true;

	function extractDataAttribute(input: string) {
		// Use a regular expression to extract the value of the `data` attribute
		const match = input.match(/data="([^"]*)"/);
		// Check if a match was found and return the first captured group
		return match ? match[1] : null;
	}

	$: id = extractDataAttribute(token.text);

</script>

{#if id}
	{#each toolCalls as toolCall}
		{#if toolCall.id === id}
			<div class="tool-call-container">
				<div class="tool-call-header">
				{#if toolCall.isLoading}
					<div class="spinner"></div>
				{:else}
					<Plus className="size-5" />
				{/if}
					<strong>{toolCall.name}</strong> [{toolCall.id}]
				</div>
				<div class="tool-call-body">
					<div class="tool-call-args">
						<ul>
							{#each Object.entries(toolCall.args) as [key, value]}
								<li><strong>{key}:</strong> {JSON.stringify(value)}</li>
							{/each}
						</ul>
					</div>
					{#if !toolCall.isLoading}
						{#if toolCall.output}
							<div class="tool-call-output">
								<strong>Output:</strong> {JSON.stringify(toolCall.output)}
							</div>
						{/if}
					{/if}
				</div>
			</div>
		{/if}
	{/each}
{/if}

