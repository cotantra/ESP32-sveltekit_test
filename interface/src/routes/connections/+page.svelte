<script lang="ts">
	import type { PageData } from './$types';
	import MQTT from './MQTT.svelte';
	import NTP from './NTP.svelte';
	import { features } from '$lib/stores/features';

	export let data: PageData;
</script>

<div class="mx-8 my-8 flex flex-col gap-4">
	{#if $features.mqtt}
		<MQTT mqttSettings={data.mqttSettings} mqttStatus={data.mqttStatus} />
	{/if}

	{#if $features.ntp}
		<NTP ntpSettings={data.ntpSettings} ntpStatus={data.ntpStatus} />
	{/if}
</div>
