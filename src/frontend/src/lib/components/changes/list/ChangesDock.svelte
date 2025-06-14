<script lang="ts">
	import { isEmptyString, isNullish, nonNullish } from '@dfinity/utils';
	import Change from '$lib/components/changes/list/Change.svelte';
	import ChangesDockLoader from '$lib/components/changes/list/ChangesDockLoader.svelte';
	import ChangesFilter from '$lib/components/changes/list/ChangesFilter.svelte';
	import ChangesMore from '$lib/components/changes/list/ChangesMore.svelte';
	import { openSatellitesProposals } from '$lib/derived/proposals.derived';
	import { satelliteStore } from '$lib/derived/satellite.derived';
	import { i18n } from '$lib/stores/i18n.store';
	import { satelliteName } from '$lib/utils/satellite.utils';

	let innerWidth = $state(0);

	let colspan = $derived(innerWidth >= 768 ? 5 : innerWidth >= 576 ? 4 : 3);

	let satelliteId = $derived($satelliteStore?.satellite_id.toText());

	let proposals = $derived(
		nonNullish(satelliteId) ? $openSatellitesProposals[satelliteId] : undefined
	);
</script>

<svelte:window bind:innerWidth />

<ChangesDockLoader>
	<div class="table-container">
		<table>
			<thead>
				<tr>
					<th {colspan}>
						<div class="actions">
							<ChangesFilter />
							{#if nonNullish($satelliteStore)}
								<ChangesMore satelliteId={$satelliteStore.satellite_id} />
								{satelliteName($satelliteStore)} ({$satelliteStore.satellite_id.toText()})
							{/if}
						</div>
					</th>
				</tr>
				<tr>
					<th class="tools"></th>
					<th> {$i18n.changes.id} </th>
					<th class="hash"> {$i18n.changes.hash} </th>
					<th> {$i18n.changes.type} </th>
					<th class="created_at"> {$i18n.sort.created_at} </th>
				</tr>
			</thead>

			<tbody>
				{#if nonNullish($satelliteStore)}
					{#each proposals ?? [] as proposal (proposal[0])}
						<Change {proposal} satellite={$satelliteStore} />
					{/each}
				{/if}

				{#if isEmptyString(satelliteId)}
					<tr
						><td {colspan}><span class="no-upgrade">{$i18n.changes.select_a_satellite}</span></td
						></tr
					>
				{:else if isNullish(proposals) || proposals.length === 0}
					<tr><td {colspan}><span class="no-upgrade">{$i18n.changes.no_changes}</span></td></tr>
				{/if}
			</tbody>
		</table>
	</div>
</ChangesDockLoader>

<style lang="scss">
	@use '../../../styles/mixins/media';

	table {
		table-layout: auto;
	}

	.hash {
		display: none;

		@include media.min-width(small) {
			display: table-cell;
		}
	}

	.created_at {
		display: none;

		@include media.min-width(medium) {
			display: table-cell;
		}
	}

	.tools {
		width: 80px;
	}

	.no-upgrade {
		white-space: normal;
	}

	.actions {
		display: flex;
		gap: var(--padding-1_5x);
		padding: var(--padding) 0;
	}
</style>
