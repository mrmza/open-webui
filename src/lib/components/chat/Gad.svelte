<script lang="ts">
	import { toast } from 'svelte-sonner';
	import { onMount, getContext, createEventDispatcher } from 'svelte';
	const i18n = getContext('i18n');
	const dispatch = createEventDispatcher();

	import { chatId, showGad, showControls } from '$lib/stores';
	import XMark from '../icons/XMark.svelte';
	import { copyToClipboard, createMessagesList } from '$lib/utils';
	import ArrowsPointingOut from '../icons/ArrowsPointingOut.svelte';
	import Tooltip from '../common/Tooltip.svelte';
	import SvgPanZoom from '../common/SVGPanZoom.svelte';
	import ArrowLeft from '../icons/ArrowLeft.svelte';

	export let overlay = false;
	export let history;
	let messages = [];

	let contents: Array<{ type: string; content: string }> = [];
	let selectedContentIdx = 0;

	let copied = false;
	let iframeElement: HTMLIFrameElement;
    console.log('gad_service');

    $: if (history) {
		messages = createMessagesList(history, history.currentId);
		getGadDoc();
	} else {
		messages = [];
		getGadDoc();
	}

    const getGadDoc = () => {
		contents = [];
		messages.forEach((message) => {
			if (message?.role !== 'user' && message?.content) {
				const codeBlockContents = message.content.match(/```[\s\S]*?```/g);
				let codeBlocks = [];

				if (codeBlockContents) {
					codeBlockContents.forEach((block) => {
						const lang = block.split('\n')[0].replace('```', '').trim().toLowerCase();
						const code = block.replace(/```[\s\S]*?\n/, '').replace(/```$/, '');
						codeBlocks.push({ lang, code });
					});
				}

				let gadContent = '';
				
				codeBlocks.forEach((block) => {
					const { lang, code } = block;

					if (lang === 'gad') {

						gadContent += code + '\n';
					}
				});

				if (gadContent) {
                    console.log('gadContent');
					const renderedContent = `
                        <!DOCTYPE html>
                        <html lang="en">
                        <head>
                            <meta charset="UTF-8">
                            <meta name="viewport" content="width=device-width, initial-scale=1.0">
                            <title>KYC Document</title>
                            <style>
                                body {
                                    font-family: Arial, sans-serif;
                                    background-color: #f4f4f9;
                                    margin: 5px;
                                    padding: 5px;
                                }
                                .container {
                                    max-width: 900px;
                                    margin: 20px auto;
                                    padding: 20px;
                                    background: white;
                                    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
                                    border-radius: 8px;
                                }
                                h1, h2 {
                                    text-align: center;
                                    color: #333;
                                }
                                p {
                                    color: #555;
                                    line-height: 1.6;
                                }
                                .section {
                                    margin-bottom: 20px;
                                }
                                .section-title {
                                    font-size: 18px;
                                    font-weight: bold;
                                    color: #333;
                                    margin-bottom: 10px;
                                    border-bottom: 2px solid #ddd;
                                    padding-bottom: 5px;
                                }
                                .field {
                                    margin: 5px 0;
                                }
                                .field label {
                                    font-weight: bold;
                                    display: inline-block;
                                    width: 180px;
                                }
                                .field span {
                                    color: #333;
                                }
                            </style>
                        </head>
                        <body>
                            <div class="container">
                                <h1>KYC Document</h1>
                                <label>${gadContent}</label>
                            </div>
                        </body>
                        </html>
                    `;
					contents = [...contents, { type: 'iframe', content: renderedContent }];
				}
			}
		});

		if (contents.length === 0) {
			showControls.set(false);
			showGad.set(false);
		}

		selectedContentIdx = contents ? contents.length - 1 : 0;
	};


	function navigateContent(direction: 'prev' | 'next') {
		console.log(selectedContentIdx);

		selectedContentIdx =
			direction === 'prev'
				? Math.max(selectedContentIdx - 1, 0)
				: Math.min(selectedContentIdx + 1, contents.length - 1);

		console.log(selectedContentIdx);
	}

	const iframeLoadHandler = () => {
		iframeElement.contentWindow.addEventListener(
			'click',
			function (e) {
				const target = e.target.closest('a');
				if (target && target.href) {
					e.preventDefault();
					const url = new URL(target.href, iframeElement.baseURI);
					if (url.origin === window.location.origin) {
						iframeElement.contentWindow.history.pushState(
							null,
							'',
							url.pathname + url.search + url.hash
						);
					} else {
						console.log('External navigation blocked:', url.href);
					}
				}
			},
			true
		);

		// Cancel drag when hovering over iframe
		iframeElement.contentWindow.addEventListener('mouseenter', function (e) {
			e.preventDefault();
			iframeElement.contentWindow.addEventListener('dragstart', (event) => {
				event.preventDefault();
			});
		});
	};

	const showFullScreen = () => {
		if (iframeElement.requestFullscreen) {
			iframeElement.requestFullscreen();
		} else if (iframeElement.webkitRequestFullscreen) {
			iframeElement.webkitRequestFullscreen();
		} else if (iframeElement.msRequestFullscreen) {
			iframeElement.msRequestFullscreen();
		}
	};

	onMount(() => {});

</script>

<div class=" w-full h-full relative flex flex-col bg-gray-50 dark:bg-gray-850">
	<div class="w-full h-full flex-1 relative">
		{#if overlay}
			<div class=" absolute top-0 left-0 right-0 bottom-0 z-10"></div>
		{/if}

		<div class="absolute pointer-events-none z-50 w-full flex items-center justify-start p-4">
			<button
				class="self-center pointer-events-auto p-1 rounded-full bg-white dark:bg-gray-850"
				on:click={() => {
					showGad.set(false);
				}}
			>
				<ArrowLeft className="size-3.5  text-gray-900 dark:text-white" />
			</button>
		</div>

		<div class=" absolute pointer-events-none z-50 w-full flex items-center justify-end p-4">
			<button
				class="self-center pointer-events-auto p-1 rounded-full bg-white dark:bg-gray-850"
				on:click={() => {
					dispatch('close');
					showControls.set(false);
					showGad.set(false);
				}}
			>
				<XMark className="size-3.5 text-gray-900 dark:text-white" />
			</button>
		</div>

		<div class="flex-1 w-full h-full">
			<div class=" h-full flex flex-col">
				{#if contents.length > 0}
					<div class="max-w-full w-full h-full">
						{#if contents[selectedContentIdx].type === 'iframe'}
							<iframe
								bind:this={iframeElement}
								title="Content"
								srcdoc={contents[selectedContentIdx].content}
								class="w-full border-0 h-full rounded-none"
								sandbox="allow-scripts allow-forms allow-same-origin"
								on:load={iframeLoadHandler}
							></iframe>
						{:else if contents[selectedContentIdx].type === 'svg'}
							<SvgPanZoom
								className=" w-full h-full max-h-full overflow-hidden"
								svg={contents[selectedContentIdx].content}
							/>
						{/if}
					</div>
				{:else}
					<div class="m-auto font-medium text-xs text-gray-900 dark:text-white">
						{$i18n.t('No HTML, CSS, or JavaScript content found.')}
					</div>
				{/if}
			</div>
		</div>
	</div>

	{#if contents.length > 0}
		<div class="flex justify-between items-center p-2.5 font-primar text-gray-900 dark:text-white">
			<div class="flex items-center space-x-2">
				<div class="flex items-center gap-0.5 self-center min-w-fit" dir="ltr">
					<button
						class="self-center p-1 hover:bg-black/5 dark:hover:bg-white/5 dark:hover:text-white hover:text-black rounded-md transition disabled:cursor-not-allowed"
						on:click={() => navigateContent('prev')}
						disabled={contents.length <= 1}
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
							stroke-width="2.5"
							class="size-3.5"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								d="M15.75 19.5 8.25 12l7.5-7.5"
							/>
						</svg>
					</button>

					<div class="text-xs self-center dark:text-gray-100 min-w-fit">
						{$i18n.t('Version {{selectedVersion}} of {{totalVersions}}', {
							selectedVersion: selectedContentIdx + 1,
							totalVersions: contents.length
						})}
					</div>

					<button
						class="self-center p-1 hover:bg-black/5 dark:hover:bg-white/5 dark:hover:text-white hover:text-black rounded-md transition disabled:cursor-not-allowed"
						on:click={() => navigateContent('next')}
						disabled={contents.length <= 1}
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
							stroke-width="2.5"
							class="size-3.5"
						>
							<path stroke-linecap="round" stroke-linejoin="round" d="m8.25 4.5 7.5 7.5-7.5 7.5" />
						</svg>
					</button>
				</div>
			</div>

			<div class="flex items-center gap-1">
				<button
					class="copy-code-button bg-none border-none text-xs bg-gray-50 hover:bg-gray-100 dark:bg-gray-850 dark:hover:bg-gray-800 transition rounded-md px-1.5 py-0.5"
					on:click={() => {
						copyToClipboard(contents[selectedContentIdx].content);
						copied = true;

						setTimeout(() => {
							copied = false;
						}, 2000);
					}}>{copied ? $i18n.t('Copied') : $i18n.t('Copy')}</button
				>

				{#if contents[selectedContentIdx].type === 'iframe'}
					<Tooltip content={$i18n.t('Open in full screen')}>
						<button
							class=" bg-none border-none text-xs bg-gray-50 hover:bg-gray-100 dark:bg-gray-850 dark:hover:bg-gray-800 transition rounded-md p-0.5"
							on:click={showFullScreen}
						>
							<ArrowsPointingOut className="size-3.5" />
						</button>
					</Tooltip>
				{/if}
			</div>
		</div>
	{/if}
</div>
