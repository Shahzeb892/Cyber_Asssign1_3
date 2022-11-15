<script lang="ts">
  import { tick } from 'svelte';
  import { get } from 'svelte/store';

  import {
    CancelOrProceedButtonPair,
    ControlledModal,
    LabeledInput,
    TextInput,
  } from '@mathesar-component-library';
  import Form from '@mathesar/components/Form.svelte';
  import FormField from '@mathesar/components/FormField.svelte';
  import SelectProcessedColumns from '@mathesar/components/SelectProcessedColumns.svelte';
  import { scrollBasedOnSelection } from '@mathesar/components/sheet';
  import {
    getTabularDataStoreFromContext,
    type ProcessedColumn,
  } from '@mathesar/stores/table-data';
  import {
    getTableFromStoreOrApi,
    moveColumns,
    splitTable,
    tables as tablesDataStore,
  } from '@mathesar/stores/tables';
  import { toast } from '@mathesar/stores/toast';
  import { getErrorMessage } from '@mathesar/utils/errors';
  import type { LinkedTable } from './columnExtractionTypes';
  import { getLinkedTables } from './columnExtractionUtils';
  import type { ExtractColumnsModalController } from './ExtractColumnsModalController';
  import SelectLinkedTable from './SelectLinkedTable.svelte';

  const tabularData = getTabularDataStoreFromContext();

  export let controller: ExtractColumnsModalController;

  let linkedTable: LinkedTable | undefined = undefined;
  let tableName = '';
  let newFkColumnName = '';

  $: ({ processedColumns, constraintsDataStore, selection } = $tabularData);
  $: ({ constraints } = $constraintsDataStore);
  $: availableColumns = [...$processedColumns.values()];
  $: ({ targetType, columns, isOpen } = controller);
  $: canProceed = true;
  $: proceedButtonLabel =
    $targetType === 'existingTable' ? 'Move Columns' : 'Create Table';
  $: linkedTables = getLinkedTables({
    constraints,
    columns: $processedColumns,
    tables: $tablesDataStore.data,
  });

  function init() {
    tableName = '';
    newFkColumnName = '';
  }

  function handleTableNameUpdate() {
    newFkColumnName = tableName;
  }

  function handleColumnsChange(_columns: ProcessedColumn[]) {
    if (!$isOpen) {
      return;
    }
    if (_columns.length === 0) {
      // If the user clears the chosen columns, we don't want to update the
      // selected cells because it would mean that no columns are selected. When
      // no columns are selected, the column pane of the table inspector closes,
      // unmounting this component.
      return;
    }
    selection.intersectSelectedRowsWithGivenColumns(_columns);
  }
  $: handleColumnsChange($columns);

  async function handleSave() {
    const followUps: Promise<unknown>[] = [];
    try {
      if ($targetType === 'existingTable') {
        const targetTableId = linkedTable?.table.id;
        if (!targetTableId) {
          throw new Error('No target table selected');
        }
        await moveColumns(
          $tabularData.id,
          $columns.map((c) => c.id),
          targetTableId,
        );
      } else {
        const response = await splitTable(
          $tabularData.id,
          $columns.map((c) => c.id),
          tableName,
        );
        followUps.push(getTableFromStoreOrApi(response.extracted_table));
      }
      followUps.push($tabularData.refresh());
      await Promise.all(followUps);
      if ($targetType === 'newTable') {
        // We ase using `get(processedColumns)` instead of `$processedColumns`
        // because the store gets updated when the above promises settle, and
        // for some reason Svelte doesn't seem to dispatch that update to the
        // store when we're using the dollar syntax here. There may be a cleaner
        // approach.
        const allColumns = [...get(processedColumns).values()];
        // Here we assume that the new column will be positioned at the end. We
        // will need to modify this logic when we position the new column where
        // the old columns were.
        const newFkColumn = allColumns.slice(-1)[0];
        selection.toggleColumnSelection(newFkColumn);
        await tick();
        scrollBasedOnSelection();
      }
      toast.success('Successfully extracted columns.');
      controller.close();
    } catch (e) {
      toast.error(getErrorMessage(e));
    }
  }
</script>

<ControlledModal {controller} on:open={init}>
  <span slot="title">
    {#if $targetType === 'existingTable'}
      Move Columns to Linked Table
    {:else}
      New Linked Table From Columns
    {/if}
  </span>

  <Form>
    {#if $targetType === 'newTable'}
      <FormField>
        <LabeledInput layout="stacked">
          <span slot="label">Name of New Table</span>
          <TextInput bind:value={tableName} on:input={handleTableNameUpdate} />
        </LabeledInput>
      </FormField>

      <!--
        TODO Uncomment and implement when
        https://github.com/centerofci/mathesar/issues/1434 is done
      -->
      <!-- <FormField>
        <LabeledInput layout="stacked">
          <span slot="label">Name of New Linking Column In This Table</span>
          <TextInput bind:value={newFkColumnName} />
        </LabeledInput>
      </FormField> -->
    {/if}

    {#if $targetType === 'existingTable'}
      <FormField>
        <LabeledInput layout="stacked">
          <span slot="label" class="label">
            <span class="title">Linked Table</span>
            <span class="help" />
          </span>
          <SelectLinkedTable {linkedTables} bind:value={linkedTable} />
        </LabeledInput>
      </FormField>
    {/if}

    <FormField>
      <LabeledInput layout="stacked">
        <span slot="label" class="label">
          <span class="title">Columns to Move</span>
          <span class="help">
            These columns will be removed from the current table and moved to
            the linked table.
          </span>
        </span>
        <SelectProcessedColumns {availableColumns} bind:columns={$columns} />
      </LabeledInput>
    </FormField>
  </Form>

  <CancelOrProceedButtonPair
    slot="footer"
    onProceed={handleSave}
    onCancel={() => controller.close()}
    proceedButton={{ label: proceedButtonLabel }}
    {canProceed}
  />
</ControlledModal>

<style>
  .label {
    display: block;
  }
  .title {
    display: block;
  }
  .help {
    display: block;
    font-size: var(--text-size-small);
    color: var(--color-text-muted);
    margin-top: 0.5rem;
  }
</style>