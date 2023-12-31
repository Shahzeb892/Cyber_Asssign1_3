<script lang="ts">
  import {
    Chip,
    Icon,
    SpinnerButton,
    Button,
    Help,
    Tooltip,
  } from '@mathesar-component-library';
  import {
    iconUser,
    iconDeleteMajor,
    iconDatabase,
    iconSchema,
  } from '@mathesar/icons';
  import type { UserModel } from '@mathesar/stores/users';
  import type { UserRole } from '@mathesar/api/users';
  import {
    getDisplayNameForRole,
    getDescriptionForRole,
    getObjectWithHighestPrecedenceByRoles,
    type ObjectRoleMap,
    type AccessControlObject,
  } from '@mathesar/utils/permissions';

  export let userProfile: UserModel | undefined;
  export let userModel: UserModel;
  export let accessControlObject: AccessControlObject;
  export let getUserRoles: (user: UserModel) => ObjectRoleMap | undefined;
  export let removeAccessForUser: (user: UserModel) => Promise<void>;

  $: roleMap = getUserRoles(userModel);
  $: roles = roleMap && roleMap.size > 0 ? [...roleMap.entries()] : [];
  $: userRoleRows = (() => {
    const rows: ['admin' | AccessControlObject, UserRole][] =
      userModel.isSuperUser ? [['admin', 'manager']] : roles;
    return rows;
  })();
  $: hasMultipleRoles = userRoleRows.length > 1;
  $: objectWithHighestPrecedence = roleMap
    ? getObjectWithHighestPrecedenceByRoles(roleMap)
    : undefined;
  $: isUserEditingTheirOwnAccess = userProfile?.id === userModel.id;

  async function removeAccess() {
    await removeAccessForUser(userModel);
  }

  function isRoleInherited(aclObject: AccessControlObject): boolean {
    return accessControlObject !== aclObject;
  }

  function getRoleStatusText(
    aclObject: AccessControlObject,
    isOverridden: boolean,
  ): string {
    const status: string[] = [];
    if (isRoleInherited(aclObject)) {
      status.push('Inherited');
    }
    if (isOverridden) {
      status.push('Overridden');
    }
    return status.length > 0 ? `(${status.join(', ')})` : '';
  }

  function getRoleHelpText(
    aclObject: AccessControlObject,
    isOverridden: boolean,
  ): string {
    const comparedAclObject = aclObject === 'database' ? 'schema' : 'database';
    const isInherited = isRoleInherited(aclObject);
    if (isInherited && isOverridden) {
      return `This access level is inherited from the ${aclObject} permissions
        and is overridden by the ${comparedAclObject} permissions.`;
    }
    if (isInherited) {
      return `This access level is inherited from the ${aclObject} permissions.`;
    }
    if (isOverridden) {
      return `This access level is overridden by the ${comparedAclObject} permissions.`;
    }
    return `This access level overrides the ${comparedAclObject} permissions.`;
  }

  function getDisabledDeleteHelpText(
    level: 'admin' | AccessControlObject,
  ): string {
    if (level === 'admin') {
      return 'Individual permissions cannot be modified for users with Admin access.';
    }
    if (isUserEditingTheirOwnAccess) {
      return 'You cannot modify your own access levels. Please contact an administrator.';
    }
    if (isRoleInherited(level)) {
      return 'This access level is inherited and cannot be removed from this panel.';
    }
    return 'This access level cannot be removed.';
  }
</script>

<div class="wrapper">
  <div class="cell has-border" style:grid-row="span {userRoleRows.length}">
    <div class="name-and-info">
      <div class="name">{userModel.username}</div>
      <div class="info">
        {#if userModel.fullName}
          <span>{userModel.fullName}</span>
        {/if}
        {#if userModel.fullName && userModel.email}
          <span class="divider" />
        {/if}
        {#if userModel.email}
          <span>{userModel.email}</span>
        {/if}
      </div>
    </div>
  </div>
  {#each userRoleRows as [level, role], index (`${level}-${role}`)}
    {@const isOverridden =
      hasMultipleRoles && objectWithHighestPrecedence !== level}
    {@const hasBorder = (hasMultipleRoles && index === 0) || !hasMultipleRoles}
    <div class="access-wrapper">
      <div
        class="cell access-level"
        class:disabled={isOverridden}
        class:has-border={hasBorder}
      >
        <div>
          {#if level === 'admin'}
            Admin Access
          {:else}
            {getDescriptionForRole(role)}
          {/if}
          {#if hasMultipleRoles && level !== 'admin'}
            {getRoleStatusText(level, isOverridden)}
            <Help>
              {getRoleHelpText(level, isOverridden)}
            </Help>
          {/if}
        </div>
      </div>
      <div
        class="cell access-level-chip"
        class:disabled={isOverridden}
        class:has-border={hasBorder}
      >
        <Chip background="var(--slate-200)" display="inline-flex">
          {#if level === 'admin'}
            <Icon {...iconUser} size="0.8em" />
            <span>Admin</span>
          {:else}
            {#if level === 'database'}
              <Icon {...iconDatabase} size="0.8em" />
            {:else if level === 'schema'}
              <Icon {...iconSchema} size="0.8em" />
            {/if}
            <span>{getDisplayNameForRole(role)}</span>
          {/if}
        </Chip>
      </div>
      <div class="cell" class:has-border={hasBorder}>
        {#if level === 'admin' || isRoleInherited(level) || isUserEditingTheirOwnAccess}
          <Tooltip>
            <Button slot="trigger" disabled>
              <Icon {...iconDeleteMajor} size="0.75em" />
            </Button>
            <span slot="content">
              {getDisabledDeleteHelpText(level)}
            </span>
          </Tooltip>
        {:else}
          <SpinnerButton
            onClick={removeAccess}
            label=""
            icon={{ ...iconDeleteMajor, size: '0.75em' }}
            appearance="outline-primary"
          />
        {/if}
      </div>
    </div>
  {/each}
</div>

<style lang="scss">
  .wrapper {
    display: contents;

    & + :global(.wrapper .cell.has-border) {
      border-top: 1px solid var(--slate-200);
    }

    .cell {
      height: 100%;
      display: flex;
      align-items: center;
      padding: var(--size-xx-small);

      &.disabled {
        color: var(--color-text-muted);
      }
    }

    .name-and-info {
      align-self: start;

      .name {
        font-weight: 500;
      }
      .info {
        display: flex;
        align-items: center;
        color: var(--slate-500);

        .divider {
          display: inline-block;
          margin: 0 0.4rem;
          width: 0.25rem;
          height: 0.25rem;
          border-radius: 50%;
          background: var(--slate-800);
        }
      }
    }
    .access-level-chip {
      justify-content: end;

      :global(.chip) {
        min-width: 4.15rem;
        justify-content: center;
      }

      :global(.chip + .chip) {
        margin-left: 0.2rem;
      }
    }
    .access-wrapper {
      display: contents;

      & + :global(.access-wrapper .cell) {
        padding-top: 0;
      }
    }
  }
</style>
