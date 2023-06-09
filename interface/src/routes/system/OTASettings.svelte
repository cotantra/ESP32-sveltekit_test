<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import { openModal, closeModal } from 'svelte-modals';
	import { user, security } from '$lib/stores/user';
	import Spinner from '$lib/components/Spinner.svelte';
	import ConfirmDialog from '$lib/components/ConfirmDialog.svelte';
	import InputPassword from '$lib/components/InputPassword.svelte';
	import SettingsCard from '$lib/components/SettingsCard.svelte';
	import { features } from '$lib/stores/features';
	import OTA from '~icons/tabler/refresh-alert';
	import Warning from '~icons/tabler/alert-triangle';
	import Cancel from '~icons/tabler/x';

	type otaSettingsType = {
		enabled: boolean;
		port: number;
		password: string;
	};

	let otaSettings: otaSettingsType;

	async function getOTASettings() {
		try {
			const otaSettingsRes = await fetch('/rest/otaSettings', {
				method: 'GET',
				headers: {
					Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
					'Content-Type': 'application/json'
				}
			});
			otaSettings = await otaSettingsRes.json();
			return otaSettings;
		} catch (error) {
			console.error('Error:', error);
		}
	}

	onMount(() => getOTASettings());

	let errorPort = false;

	async function postOTASettings(data: otaSettingsType) {
		try {
			const response = await fetch('/rest/otaSettings', {
				method: 'POST',
				headers: {
					Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(data)
			});

			otaSettings = await response.json();
		} catch (error) {
			console.error('Error:', error);
		}
	}

	function handleSubmitOTA() {
		// Validate if port is a number and within the right range
		let port = Number(otaSettings.port);
		if (1024 < port && port <= 65536) {
			errorPort = false;

			// Submit JSON to REST API
			postOTASettings(otaSettings);
		} else {
			errorPort = true;
		}
	}

	let files: FileList;

	async function uploadBIN() {
		try {
			const formData = new FormData();
			formData.append('file', files[0]);
			const response = await fetch('/rest/uploadFirmware', {
				method: 'POST',
				headers: {
					Authorization: $security.security ? 'Bearer ' + $user.bearer_token : 'Basic',
					'Content-Type': 'application/json'
				},
				body: formData
			});
			const result = await response.json();
		} catch (error) {
			console.error('Error:', error);
		}
	}

	function confirmBinUpload() {
		openModal(ConfirmDialog, {
			title: 'Confirm Flashing the Device',
			message: 'Are you sure you want to overwrite the existing firmware with a new one?',
			labels: {
				cancel: { label: 'Abort', icon: Cancel },
				confirm: { label: 'Upload', icon: OTA }
			},
			onConfirm: () => {
				closeModal();
				uploadBIN();
			}
		});
	}
</script>

<SettingsCard open={false}>
	<OTA slot="icon" class="lex-shrink-0 mr-2 h-6 w-6 self-end rounded-full" />
	<span slot="title">OTA Firmware Update</span>
	{#if $features.ota}
		{#await getOTASettings()}
			<Spinner />
		{:then nothing}
			<label class="label cursor-pointer justify-start gap-4">
				<input
					type="checkbox"
					bind:checked={otaSettings.enabled}
					class="checkbox checkbox-primary"
				/>
				<span class="">Enable Remote OTA Updates?</span>
			</label>

			<form class="form-control w-full" on:submit|preventDefault={handleSubmitOTA} novalidate>
				<label class="label" for="port">
					<span class="label-text text-md">Port</span>
				</label>
				<input
					type="number"
					min="1025"
					max="65536"
					class="input input-bordered invalid:border-error w-full invalid:border-2"
					bind:value={otaSettings.port}
					id="port"
					required
				/>
				<label for="port" class="label"
					><span class="label-text-alt text-error {errorPort ? '' : 'hidden'}"
						>Port number must be between 1025 and 65536</span
					></label
				>
				<label class="label" for="pwd">
					<span class="label-text text-md">Password</span>
				</label>
				<InputPassword bind:value={otaSettings.password} id="pwd" />
				<div class="mt-6 flex justify-end">
					<button class="btn btn-primary" type="submit">Apply Settings</button>
				</div>
			</form>
		{/await}
	{/if}

	{#if $features.ota && $features.upload_firmware}
		<div class="divider" />
	{/if}

	{#if $features.upload_firmware}
		<span class="text-lg font-semibold">Upload Firmware</span>
		<div class="alert alert-warning shadow-lg">
			<div>
				<Warning class="h-6 w-6 flex-shrink-0" />
				<span>Uploading a new firmware (.bin) file will replace the existing firmware.</span>
			</div>
		</div>

		<input
			type="file"
			id="binFile"
			class="file-input file-input-bordered file-input-secondary mt-4 w-full"
			bind:files
			accept=".bin"
			on:change={confirmBinUpload}
		/>
	{/if}
</SettingsCard>
