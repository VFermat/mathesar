<script lang="ts">
  import {
    faSortAmountDown,
    faSortAmountDownAlt,
    faThList,
    faSpinner,
  } from '@fortawesome/free-solid-svg-icons';
  import { Checkbox, Dropdown, Icon } from '@mathesar-components';
  import type { ConstraintsDataStore } from '@mathesar/stores/table-data/constraints';
  import type {
    Meta,
    Column,
    SortOption,
    GroupOption,
    ColumnPosition,
    ColumnsDataStore,
  } from '@mathesar/stores/table-data/types';

  export let columnPosition: ColumnPosition;
  export let column: Column;
  export let meta: Meta;
  export let constraintsDataStore: ConstraintsDataStore;
  export let columnsDataStore: ColumnsDataStore;
  
  let isRequestingToggleAllowDuplicates = false;
  let isRequestingToggleAllowNull = false;
  let menuIsOpen = false;
  
  $: ({ sort, group } = meta);
  $: sortDirection = ($sort as SortOption)?.get(column.name);
  $: hasGrouping = ($group as GroupOption)?.has(column.name);

  $: allowsDuplicatesStore = constraintsDataStore.columnAllowsDuplicates(column);
  $: allowsDuplicates = $allowsDuplicatesStore as boolean;
  $: allowsNull = column.nullable;

  function closeMenu() {
    menuIsOpen = false;
  }
  
  function handleSort(order: 'asc' | 'desc') {
    if (sortDirection === order) {
      meta.removeSort(column.name);
    } else {
      meta.addUpdateSort(column.name, order);
    }
    closeMenu();
  }

  function toggleGroup() {
    if (hasGrouping) {
      meta.removeGroup(column.name);
    } else {
      meta.addGroup(column.name);
    }
    closeMenu();
  }

  async function toggleAllowDuplicates() {
    isRequestingToggleAllowDuplicates = true;
    try {
      const isUnique = await constraintsDataStore.updateUniquenessOfColumn(column, (u) => !u);
      const msg = `Column "${column.name}" will ${isUnique ? 'no longer ' : ''}allow duplicates.`;
      console.log(msg); // TODO display success toast message: msg
      closeMenu();
    } catch (error) {
      const msg = `Unable to update "Allow Duplicates" of column "${column.name}". ${error.message as string}.`;
      console.log(msg); // TODO display error toast message
    } finally {
      isRequestingToggleAllowDuplicates = false;
    }
  }

  async function toggleAllowNull() {
    isRequestingToggleAllowNull = true;
    try {
      const newAllowsNull = !allowsNull;
      await columnsDataStore.setNullabilityOfColumn(column, newAllowsNull);
      const msg = `Column "${column.name}" will ${newAllowsNull ? '' : 'no longer '}allow NULL.`;
      console.log(msg); // TODO display success toast message: msg
      closeMenu();
    } catch (error) {
      const msg = `Unable to update "Allow NULL" of column "${column.name}". ${error.message as string}.`;
      console.log(msg); // TODO display error toast message
    } finally {
      isRequestingToggleAllowNull = false;
    }
  }
</script>

<div class="cell" style="width:{columnPosition?.width || 0}px;
      left:{(columnPosition?.left || 0)}px;">
  <span class="type">
    {#if column.type === 'INTEGER'}
      #
    {:else if column.type === 'VARCHAR'}
      T
    {:else}
      i
    {/if}
  </span>
  <span class="name">{column.name}</span>

  <Dropdown
    triggerClass="opts"
    triggerAppearance="plain"
    contentClass="table-opts-content"
    bind:isOpen={menuIsOpen}
  >
    <svelte:fragment slot="content">
      <ul>
        <li on:click={() => handleSort('asc')}>
          <Icon class="opt" data={faSortAmountDownAlt}/>
          <span>
            {#if sortDirection === 'asc'}
              Remove asc sort
            {:else}
              Sort Ascending
            {/if}
          </span>
        </li>
        <li on:click={() => handleSort('desc')}>
          <Icon class="opt" data={faSortAmountDown}/>
          <span>
            {#if sortDirection === 'desc'}
              Remove desc sort
            {:else}
              Sort Descending
            {/if}
          </span>
        </li>
        <li on:click={toggleGroup}>
          <Icon class="opt" data={faThList}/>
          <span>
            {#if hasGrouping}
              Remove grouping
            {:else}
              Group by column
            {/if}
          </span>
        </li>
        <!--
          TODO Once we have a DropdownMenu component, make this option
          disabled if the column is a primary key.
        -->
        <li on:click={toggleAllowNull}>
          {#if isRequestingToggleAllowNull}
            <Icon class="opt" data={faSpinner} spin={true}/>
          {:else}
            <span class="opt"><Checkbox checked={allowsNull} /></span>
          {/if}
          <span>Allow NULL</span>
        </li>
        <!--
          TODO Once we have a DropdownMenu component, make this option
          disabled if the column is a primary key.
        -->
        <li on:click={toggleAllowDuplicates}>
          {#if isRequestingToggleAllowDuplicates}
            <Icon class="opt" data={faSpinner} spin={true}/>
          {:else}
            <span class="opt"><Checkbox checked={allowsDuplicates} /></span>
          {/if}
          <span>Allow Duplicates</span>
        </li>
      </ul>
    </svelte:fragment>
  </Dropdown>
</div>
