<script lang="ts">
	import { closeModal } from 'svelte-modals';
	import { fly } from 'svelte/transition';
	import { user, security } from '$lib/stores/user';
	import WiFi from '~icons/tabler/wifi';
	import Network from '~icons/tabler/router';
	import AP from '~icons/tabler/access-point';
	import Cancel from '~icons/tabler/x';
	import Reload from '~icons/tabler/reload';
	import { onMount } from 'svelte';

	// provided by <Modals />
	export let isOpen;
	export let storeNetwork;

	const encryptionType = [
		'Open',
		'WEP',
		'WPA PSK',
		'WPA2 PSK',
		'WPA WPA2 PSK',
		'WPA2 Enterprise',
		'WPA3 PSK',
		'WPA2 WPA3 PSK',
		'WAPI PSK'
	];

	let listOfNetworks = [];

	let scanActive = false;

	async function scanNetworks() {
		scanActive = true;
		const scan = await fetch('/rest/scanNetworks', {
			method: 'GET',
			headers: {
				Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
				'Content-Type': 'application/json'
			}
		});
		const response = await fetch('/rest/listNetworks', {
			method: 'GET',
			headers: {
				Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
				'Content-Type': 'application/json'
			}
		});
		const result = await response.json();
		listOfNetworks = result.networks;
		scanActive = false;
		return;
	}

	onMount(() => {
		scanNetworks();
	});
</script>

{#if isOpen}
	<div
		role="dialog"
		class="pointer-events-none fixed inset-0 z-50 flex items-center justify-center overflow-y-auto"
		transition:fly={{ y: 50 }}
		on:introstart
		on:outroend
	>
		<div
			class="bg-base-100 shadow-secondary/30 pointer-events-auto flex min-w-fit max-w-md flex-col justify-between rounded-xl p-4 shadow-lg"
		>
			<h2 class="text-base-content text-start text-2xl font-bold">Scan Networks</h2>
			<div class="divider my-2" />
			<div>
				{#if scanActive}<div
						class="bg-base-100 flex h-full w-full flex-col items-center justify-center p-6"
					>
						<AP class="text-secondary h-32 w-32 animate-ping stroke-2" />
						<p class="mt-8 text-2xl">Scanning ...</p>
					</div>
				{:else}
					<ul class="menu">
						{#each listOfNetworks as network, i}
							<li>
								<!-- svelte-ignore a11y-click-events-have-key-events -->
								<div
									class="bg-base-200 my-1 flex items-center space-x-3 rounded-md hover:scale-[1.02] active:scale-[0.98]"
									on:click={() => {
										storeNetwork(network.ssid);
									}}
								>
									<div class="mask mask-hexagon bg-primary h-auto w-10 shrink-0">
										<Network class="text-primary-content h-auto w-full scale-75" />
									</div>
									<div>
										<div class="font-bold">{network.ssid}</div>
										<div class="text-sm opacity-75">
											Security: {encryptionType[network.encryption_type]}, Channel: {network.channel}
										</div>
									</div>
									<div class="flex-grow" />
									<div class="indicator">
										<span
											class="indicator-item indicator-start badge badge-accent badge-outline badge-xs"
											>{network.rssi} dBm</span
										>
										<WiFi class="text-base-content h-auto w-10 opacity-50" />
									</div>
								</div>
							</li>
						{/each}
					</ul>
				{/if}
			</div>
			<div class="divider my-2" />
			<div class="flex flex-wrap justify-end gap-2">
				<button
					class="btn btn-primary inline-flex flex-none items-center"
					disabled={scanActive}
					on:click={scanNetworks}><Reload class="mr-2 h-5 w-5" /><span>Scan again</span></button
				>

				<div class="flex-grow" />
				<button
					class="btn btn-warning text-warning-content inline-flex flex-none items-center"
					on:click={closeModal}><Cancel class="mr-2 h-5 w-5" /><span>Cancel</span></button
				>
			</div>
		</div>
	</div>
{/if}
