<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import type { CellValueFormatter } from '@mathesar/components/cell-fabric/utils';
  import CellValue from '@mathesar/components/CellValue.svelte';
  import CellWrapper from './CellWrapper.svelte';
  import type { CellTypeProps, HorizontalAlignment } from './typeDefinitions';

  const dispatch = createEventDispatcher();

  type Value = $$Generic;
  type Props = CellTypeProps<Value>;

  export let isActive: Props['isActive'];
  export let isSelectedInRange: Props['isSelectedInRange'];
  export let value: Props['value'];
  export let disabled: Props['disabled'];
  export let multiLineTruncate = false;
  export let formatValue: CellValueFormatter<Value> | undefined = undefined;
  export let horizontalAlignment: HorizontalAlignment | undefined = undefined;

  let cellRef: HTMLElement;
  let isEditMode = false;

  function setModeToEdit() {
    if (!disabled) {
      isEditMode = true;
    }
  }

  function resetEditMode() {
    isEditMode = false;
  }

  function handleKeyDown(e: KeyboardEvent) {
    switch (e.key) {
      case 'Enter':
        if (isEditMode) {
          resetEditMode();
        } else {
          setModeToEdit();
        }
        // Preventing default behaviour here. Interesting problem: If this is
        // not prevented, the textarea gets a new line break. Needs more digging
        // down.
        e.preventDefault();
        break;
      case 'Escape':
        setModeToEdit();
        break;
      case 'Tab':
        resetEditMode();
        dispatch('movementKeyDown', {
          originalEvent: e,
          key: e.key,
        });
        break;
      case 'ArrowLeft':
      case 'ArrowRight':
      case 'ArrowDown':
      case 'ArrowUp':
        if (!isEditMode) {
          dispatch('movementKeyDown', {
            originalEvent: e,
            key: e.key,
          });
        }
        break;
      default:
        break;
    }
  }

  function dispatchUpdate() {
    dispatch('update', {
      value,
    });
  }

  function handleInputKeydown(e: KeyboardEvent) {
    switch (e.key) {
      case 'Enter':
      case 'Escape':
      case 'Tab':
        dispatchUpdate();
        break;
      default:
        break;
    }
  }
  let hasInitialValue = false
  let typeOfTheValue = typeof value
  let defaultValue = typeOfTheValue==="number" ? 0 : '';
  console.log(defaultValue)
  let initialValue = hasInitialValue===false && value!=undefined && value!=null ? value : defaultValue;
  function escKeydown(e: KeyboardEvent){
    hasInitialValue = true
    if(e.key==='Escape'){
      let typeNumber = typeof value;
      value = typeNumber!=='number' && initialValue!==null && initialValue!==undefined ? initialValue : 0;
    }
    if(e.key==='Enter'){
      initialValue = value!==undefined && value!=null ? value : defaultValue;
    }
    console.log("value :",value)
    console.log("initialValue :",initialValue)
    console.log("defaultValue :",defaultValue)
  }

  function handleInputBlur() {
    dispatchUpdate();
    resetEditMode();
  }
</script>

<CellWrapper
  {isActive}
  {isSelectedInRange}
  {disabled}
  bind:element={cellRef}
  on:dblclick={setModeToEdit}
  on:keydown={handleKeyDown}
  on:mousedown={() => dispatch('activate')}
  on:mouseenter
  mode={isEditMode ? 'edit' : 'default'}
  {multiLineTruncate}
  {horizontalAlignment}
>
  {#if isEditMode}
    <slot {handleInputBlur} {handleInputKeydown} {escKeydown} />
  {:else}
    <div
      class="content"
      class:nowrap={!isActive}
      class:truncate={isActive && multiLineTruncate}
    >
      <slot name="content" {value} {formatValue}>
        <CellValue {value} {formatValue} />
      </slot>
    </div>
  {/if}
</CellWrapper>

<style lang="scss">
  .content {
    overflow: hidden;
    position: relative;
    text-overflow: ellipsis;

    &.nowrap {
      white-space: nowrap;
    }

    &.truncate {
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
    }
  }
</style>
