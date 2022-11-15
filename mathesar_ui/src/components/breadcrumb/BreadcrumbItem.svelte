<script lang="ts">
  import Logo from '@mathesar/components/Logo.svelte';
  import SchemaName from '@mathesar/components/SchemaName.svelte';
  import TableName from '@mathesar/components/TableName.svelte';
  import NameWithIcon from '@mathesar/components/NameWithIcon.svelte';
  import {
    getDatabasePageUrl,
    getSchemaPageUrl,
    getTablePageUrl,
  } from '@mathesar/routes/urls';
  import RecordSelectorNavigationButton from '@mathesar/systems/record-selector/RecordSelectorNavigationButton.svelte';
  import BreadcrumbLink from './BreadcrumbLink.svelte';
  import type { BreadcrumbItem } from './breadcrumbTypes';
  import EntitySelector from './EntitySelector.svelte';
  import SchemaSelector from './SchemaSelector.svelte';

  export let item: BreadcrumbItem;
</script>

<div class="container">
  {#if item.type === 'database'}
    <a href={getDatabasePageUrl(item.database.name)} class="logo-link"
      ><Logo /></a
    >
    <SchemaSelector database={item.database} />
  {:else if item.type === 'schema'}
    <BreadcrumbLink href={getSchemaPageUrl(item.database.name, item.schema.id)}>
      <SchemaName schema={item.schema} />
    </BreadcrumbLink>
    <EntitySelector database={item.database} schema={item.schema} />
  {:else if item.type === 'table'}
    <BreadcrumbLink
      href={getTablePageUrl(item.database.name, item.schema.id, item.table.id)}
    >
      <TableName table={item.table} />
    </BreadcrumbLink>
    <RecordSelectorNavigationButton table={item.table} />
  {:else if item.type === 'simple'}
    <BreadcrumbLink href={item.href}>
      {#if item.icon}
        <NameWithIcon icon={item.icon}>{item.label}</NameWithIcon>
      {:else}
        {item.label}
      {/if}
    </BreadcrumbLink>
  {/if}
</div>

<style lang="scss">
  .container {
    --icon-color: var(--brand-500);
    --name-color: var(--white);

    display: flex;
    flex-direction: row;
    align-items: center;

    > :global(* + *) {
      margin-left: 1rem;
    }
  }
  .logo-link {
    text-decoration: none;
  }
</style>