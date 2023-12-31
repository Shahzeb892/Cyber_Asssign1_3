<script lang="ts">
  import type { TableEntry } from '@mathesar/api/types/tables';
  import LayoutWithHeader from '@mathesar/layouts/LayoutWithHeader.svelte';
  import { makeSimplePageTitle } from '@mathesar/pages/pageTitleUtils';
  import { currentDbAbstractTypes } from '@mathesar/stores/abstract-types';
  import { TableStructure } from '@mathesar/stores/table-data';
  import { displayRecordSummaryAsPlainText } from '@mathesar/stores/table-data/record-summaries/recordSummaryUtils';
  import { currentTable } from '@mathesar/stores/tables';
  import RecordPageContent from './RecordPageContent.svelte';
  import RecordPageLoadingSpinner from './RecordPageLoadingSpinner.svelte';
  import type RecordStore from './RecordStore';

  export let record: RecordStore;

  $: table = $currentTable as TableEntry;
  $: tableStructure = new TableStructure({
    id: table.id,
    abstractTypesMap: $currentDbAbstractTypes.data,
  });
  $: tableStructureIsLoading = tableStructure.isLoading;
  $: recordStoreFetchRequest = record.fetchRequest;
  $: ({ summary } = record);
  $: recordStoreIsLoading = $recordStoreFetchRequest?.state === 'processing';
  $: isLoading = $tableStructureIsLoading || recordStoreIsLoading;
  $: title = recordStoreIsLoading
    ? ''
    : displayRecordSummaryAsPlainText($summary);
</script>

<svelte:head><title>{makeSimplePageTitle(title)}</title></svelte:head>

<LayoutWithHeader cssVariables={{ '--page-padding': '0' }} fitViewport>
  {#if isLoading}
    <RecordPageLoadingSpinner />
  {:else}
    <RecordPageContent {tableStructure} {record} />
  {/if}
</LayoutWithHeader>
