<script lang="ts">
  import { Collapsible } from '@mathesar-component-library';
  import { getTabularDataStoreFromContext } from '@mathesar/stores/table-data';
  import { getSelectedColumnId } from '@mathesar/stores/table-data/selection';
  import type { ProcessedColumn } from '@mathesar/stores/table-data/processedColumns';
  import RenameColumn from './RenameColumn.svelte';
  import ColumnDisplayProperties from './ColumnDisplayProperties.svelte';
  import ColumnActions from './ColumnActions.svelte';
  import ColumnOptions from './ColumnOptions.svelte';
  import ColumnType from './ColumnType.svelte';

  const tabularData = getTabularDataStoreFromContext();
  $: ({ processedColumns, selection } = $tabularData);
  $: ({ selectedCells } = selection);
  $: selectedColumnsId = $selectedCells
    .valuesArray()
    .map((cell) => getSelectedColumnId(cell));
  $: uniquelySelectedColumnsId = Array.from(new Set(selectedColumnsId));

  function getSelectedColumn(selectedColumnId: number): ProcessedColumn {
    return $processedColumns.get(selectedColumnId)!;
  }
</script>

<div class="column-mode-container">
  {#if uniquelySelectedColumnsId.length === 1}
    <Collapsible isOpen>
      <span slot="header">Column Properties</span>
      <div slot="content" class="property-container">
        <RenameColumn
          column={getSelectedColumn(uniquelySelectedColumnsId[0])}
          columnsDataStore={$tabularData.columnsDataStore}
        />
        <ColumnDisplayProperties
          column={getSelectedColumn(uniquelySelectedColumnsId[0])}
          meta={$tabularData.meta}
        />
        <ColumnOptions
          column={getSelectedColumn(uniquelySelectedColumnsId[0])}
          columnsDataStore={$tabularData.columnsDataStore}
          constraintsDataStore={$tabularData.constraintsDataStore}
        />
      </div>
    </Collapsible>

    <Collapsible isOpen>
      <span slot="header">Data Type</span>
      <div slot="content" class="actions-container">
        <ColumnType column={getSelectedColumn(uniquelySelectedColumnsId[0])} />
      </div>
    </Collapsible>

    <Collapsible isOpen>
      <span slot="header">Actions</span>
      <div slot="content" class="actions-container">
        <ColumnActions
          column={getSelectedColumn(uniquelySelectedColumnsId[0])}
          columnsDataStore={$tabularData.columnsDataStore}
        />
      </div>
    </Collapsible>
  {:else}
    <span
      >Select a single column or mulitple cells belonging to a single column to
      see column properties and actions</span
    >
  {/if}
</div>

<style>
  .column-mode-container {
    padding: 1rem 0;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .property-container {
    padding: 1rem;
  }

  .actions-container {
    padding: 1rem 0;
  }
</style>