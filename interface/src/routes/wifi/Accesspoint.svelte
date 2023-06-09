<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import InputPassword from '$lib/components/InputPassword.svelte';
	import SettingsCard from '$lib/components/SettingsCard.svelte';
	import { user, security } from '$lib/stores/user';
	import Spinner from '$lib/components/Spinner.svelte';
	import AP from '~icons/tabler/access-point';
	import MAC from '~icons/tabler/dna-2';
	import Home from '~icons/tabler/home';
	import Devices from '~icons/tabler/devices';

	type ApStatus = {
		status: number;
		ip_address: string;
		mac_address: string;
		station_num: number;
	};

	type ApSettings = {
		provision_mode: number;
		ssid: string;
		password: string;
		channel: number;
		ssid_hidden: boolean;
		max_clients: number;
		local_ip: string;
		gateway_ip: string;
		subnet_mask: string;
	};

	let apSettings: ApSettings;
	let apStatus: ApStatus;

	let formField: any;

	async function getAPStatus() {
		try {
			const response = await fetch('/rest/apStatus', {
				method: 'GET',
				headers: {
					Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
					'Content-Type': 'application/json'
				}
			});
			apStatus = await response.json();
		} catch (error) {
			console.error('Error:', error);
		}
		return apStatus;
	}

	async function getAPSettings() {
		try {
			const response = await fetch('/rest/apSettings', {
				method: 'GET',
				headers: {
					Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
					'Content-Type': 'application/json'
				}
			});
			apSettings = await response.json();
		} catch (error) {
			console.error('Error:', error);
		}
		return apSettings;
	}

	const interval = setInterval(async () => {
		getAPStatus();
	}, 5000);

	onDestroy(() => clearInterval(interval));

	let provisionMode = [
		{
			id: 0,
			text: `Always`
		},
		{
			id: 1,
			text: `When WiFi Disconnected`
		},
		{
			id: 2,
			text: `Never`
		}
	];

	let formErrors = {
		ssid: false,
		channel: false,
		max_clients: false,
		local_ip: false,
		gateway_ip: false,
		subnet_mask: false
	};

	async function postAPSettings(data: ApSettings) {
		try {
			const response = await fetch('/rest/apSettings', {
				method: 'POST',
				headers: {
					Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(data)
			});

			apSettings = await response.json();
		} catch (error) {
			console.error('Error:', error);
		}
	}

	function handleSubmitAP() {
		let valid = true;

		// Validate SSID
		if (apSettings.ssid.length < 3 || apSettings.ssid.length > 32) {
			valid = false;
			formErrors.ssid = true;
		} else {
			formErrors.ssid = false;
		}

		// Validate Channel
		let channel = Number(apSettings.channel);
		if (1 > channel || channel > 13) {
			valid = false;
			formErrors.channel = true;
		} else {
			formErrors.channel = false;
		}

		// Validate max_clients
		let maxClients = Number(apSettings.max_clients);
		if (1 > maxClients || maxClients > 8) {
			valid = false;
			formErrors.max_clients = true;
		} else {
			formErrors.max_clients = false;
		}

		// RegEx for IPv4
		const regexExp =
			/\b(?:(?:2(?:[0-4][0-9]|5[0-5])|[0-1]?[0-9]?[0-9])\.){3}(?:(?:2([0-4][0-9]|5[0-5])|[0-1]?[0-9]?[0-9]))\b/;

		// Validate gateway IP
		if (!regexExp.test(apSettings.gateway_ip)) {
			valid = false;
			formErrors.gateway_ip = true;
		} else {
			formErrors.gateway_ip = false;
		}

		// Validate Subnet Mask
		if (!regexExp.test(apSettings.subnet_mask)) {
			valid = false;
			formErrors.subnet_mask = true;
		} else {
			formErrors.subnet_mask = false;
		}

		// Validate local IP
		if (!regexExp.test(apSettings.local_ip)) {
			valid = false;
			formErrors.local_ip = true;
		} else {
			formErrors.local_ip = false;
		}

		// Submit JSON to REST API
		if (valid) {
			postAPSettings(apSettings);
		}
	}
</script>

<SettingsCard open={true}>
	<AP slot="icon" class="lex-shrink-0 mr-2 h-6 w-6 self-end" />
	<span slot="title">Access Point</span>
	<div class="w-full overflow-x-auto">
		{#await getAPStatus()}
			<Spinner />
		{:then nothing}
			<table class="table w-full">
				<tbody>
					<!-- row 1 -->
					<tr>
						<td>
							<div class="flex items-center space-x-3">
								<div
									class="mask mask-hexagon h-auto w-10 {apStatus.status === 0
										? 'bg-success'
										: 'bg-error'}"
								>
									<AP
										class="h-auto w-full scale-75 {apStatus.status === 0
											? 'text-success-content'
											: 'text-error-content'}"
									/>
								</div>
								<div>
									<div class="font-bold">Status</div>
									<div class="text-sm opacity-75">
										{apStatus.status === 0 ? 'Active' : 'Inactive'}
									</div>
								</div>
							</div>
						</td>
					</tr>
					<!-- row 2 -->
					<tr>
						<td>
							<div class="flex items-center space-x-3">
								<div class="mask mask-hexagon bg-primary h-auto w-10">
									<Home class="text-primary-content h-auto w-full scale-75" />
								</div>
								<div>
									<div class="font-bold">IP Address</div>
									<div class="text-sm opacity-75">
										{apStatus.ip_address}
									</div>
								</div>
							</div>
						</td>
					</tr>
					<!-- row 3 -->
					<tr>
						<td>
							<div class="flex items-center space-x-3">
								<div class="mask mask-hexagon bg-primary h-auto w-10">
									<MAC class="text-primary-content h-auto w-full scale-75" />
								</div>
								<div>
									<div class="font-bold">MAC Address</div>
									<div class="text-sm opacity-75">
										{apStatus.mac_address}
									</div>
								</div>
							</div>
						</td>
					</tr>
					<!-- row 4 -->
					<tr>
						<td>
							<div class="flex items-center space-x-3">
								<div class="mask mask-hexagon bg-primary h-auto w-10">
									<Devices class="text-primary-content h-auto w-full scale-75" />
								</div>
								<div>
									<div class="font-bold">AP Clients</div>
									<div class="text-sm opacity-75">
										{apStatus.station_num}
									</div>
								</div>
							</div>
						</td>
					</tr>
				</tbody>
			</table>
		{/await}
	</div>

	{#if $security.admin_required}
		<div class="collapse">
			<input type="checkbox" />
			<div class="collapse-title text-xl font-medium">Change AP Settings</div>
			<div class="collapse-content">
				{#await getAPSettings()}
					<Spinner />
				{:then nothing}
					<form
						class="grid w-full grid-cols-1 content-center gap-x-4 px-4 sm:grid-cols-2"
						on:submit|preventDefault={handleSubmitAP}
						novalidate
						bind:this={formField}
					>
						<div>
							<label class="label" for="apmode">
								<span class="label-text">Provide Access Point ...</span>
							</label>
							<select
								class="select select-bordered w-full"
								id="apmode"
								bind:value={apSettings.provision_mode}
							>
								{#each provisionMode as mode}
									<option value={mode.id}>
										{mode.text}
									</option>
								{/each}
							</select>
						</div>
						<div>
							<label class="label" for="ssid">
								<span class="label-text text-md">SSID</span>
							</label>
							<input
								type="text"
								class="input input-bordered invalid:border-error w-full invalid:border-2 {formErrors.ssid
									? 'border-error border-2'
									: ''}"
								bind:value={apSettings.ssid}
								id="ssid"
								min="2"
								max="32"
								required
							/>
							<label class="label" for="ssid">
								<span class="label-text-alt text-error {formErrors.ssid ? '' : 'hidden'}"
									>SSID must be between 2 and 32 characters long</span
								>
							</label>
						</div>

						<div>
							<label class="label" for="pwd">
								<span class="label-text text-md">Password</span>
							</label>
							<InputPassword bind:value={apSettings.password} id="pwd" />
						</div>
						<div>
							<label class="label" for="channel">
								<span class="label-text text-md">Preferred Channel</span>
							</label>
							<input
								type="number"
								min="1"
								max="13"
								class="input input-bordered invalid:border-error w-full invalid:border-2 {formErrors.channel
									? 'border-error border-2'
									: ''}"
								bind:value={apSettings.channel}
								id="channel"
								required
							/>
							<label class="label" for="channel">
								<span class="label-text-alt text-error {formErrors.channel ? '' : 'hidden'}"
									>Must be channel 1 to 13</span
								>
							</label>
						</div>

						<div>
							<label class="label" for="clients">
								<span class="label-text text-md">Max Clients</span>
							</label>
							<input
								type="number"
								min="1"
								max="8"
								class="input input-bordered invalid:border-error w-full invalid:border-2 {formErrors.max_clients
									? 'border-error border-2'
									: ''}"
								bind:value={apSettings.max_clients}
								id="clients"
								required
							/>
							<label class="label" for="clients">
								<span class="label-text-alt text-error {formErrors.max_clients ? '' : 'hidden'}"
									>Maximum 8 clients allowed</span
								>
							</label>
						</div>

						<div>
							<label class="label" for="localIP">
								<span class="label-text text-md">Local IP</span>
							</label>
							<input
								type="text"
								class="input input-bordered w-full {formErrors.local_ip
									? 'border-error border-2'
									: ''}"
								minlength="7"
								maxlength="15"
								size="15"
								bind:value={apSettings.local_ip}
								id="localIP"
								required
							/>
							<label class="label" for="localIP">
								<span class="label-text-alt text-error {formErrors.local_ip ? '' : 'hidden'}"
									>Must be a valid IPv4 address</span
								>
							</label>
						</div>

						<div>
							<label class="label" for="gateway">
								<span class="label-text text-md">Gateway IP</span>
							</label>
							<input
								type="text"
								class="input input-bordered w-full {formErrors.gateway_ip
									? 'border-error border-2'
									: ''}"
								minlength="7"
								maxlength="15"
								size="15"
								bind:value={apSettings.gateway_ip}
								id="gateway"
								required
							/>
							<label class="label" for="gateway">
								<span class="label-text-alt text-error {formErrors.gateway_ip ? '' : 'hidden'}"
									>Must be a valid IPv4 address</span
								>
							</label>
						</div>
						<div>
							<label class="label" for="subnet">
								<span class="label-text text-md">Subnet Mask</span>
							</label>
							<input
								type="text"
								class="input input-bordered w-full {formErrors.subnet_mask
									? 'border-error border-2'
									: ''}"
								minlength="7"
								maxlength="15"
								size="15"
								bind:value={apSettings.subnet_mask}
								id="subnet"
								required
							/>
							<label class="label" for="subnet">
								<span class="label-text-alt text-error {formErrors.subnet_mask ? '' : 'hidden'}"
									>Must be a valid IPv4 address</span
								>
							</label>
						</div>

						<label class="label my-auto cursor-pointer justify-start gap-4">
							<input
								type="checkbox"
								bind:checked={apSettings.ssid_hidden}
								class="checkbox checkbox-primary"
							/>
							<span class="">Hide SSID</span>
						</label>

						<div class="place-self-end">
							<button class="btn btn-primary" type="submit">Apply Settings</button>
						</div>
					</form>
				{/await}
			</div>
		</div>
	{/if}
</SettingsCard>
