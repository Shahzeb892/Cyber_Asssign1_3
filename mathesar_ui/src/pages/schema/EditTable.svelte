<script lang="ts">
  import type { TableEntry } from '@mathesar/api/types/tables';
  import type { ModalController } from '@mathesar/component-library';
  import EditTableHoc from '@mathesar/components/EditTableHOC.svelte';
  import Identifier from '@mathesar/components/Identifier.svelte';
  import NameAndDescInputModalForm from '@mathesar/components/NameAndDescInputModalForm.svelte';

  export let table: TableEntry;
  export let modalController: ModalController;
</script>

<EditTableHoc let:getNameValidationErrors let:onUpdate tableId={table.id}>
  <NameAndDescInputModalForm
    controller={modalController}
    save={(name, description) => onUpdate({ name, description })}
    {getNameValidationErrors}
    getInitialName={() => table.name ?? ''}
    getInitialDescription={() => table.description ?? ''}
  >
    <span slot="title" let:initialName>
      Edit <Identifier>{initialName}</Identifier> Table
    </span>
  </NameAndDescInputModalForm>
</EditTableHoc>
