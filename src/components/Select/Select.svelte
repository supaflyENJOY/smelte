<script>
  import { createEventDispatcher, onMount } from "svelte";
  import { quadOut, quadIn } from "svelte/easing";
  import List from "../List/List.svelte";
  import TextField from "../TextField";
  import { ClassBuilder } from "../../utils/classes.js";
  import { hideListAction } from '../../utils/hide-list-action';

  const optionsClassesDefault = "absolute left-0 bg-white rounded elevation-3 w-full z-20 dark:bg-dark-500";
  const classesDefault = "cursor-pointer relative pb-4";

  const noop = i => i;

  export let items = [];
  export let value = "";
  export const text = "";
  export let label = "";
  export let selectedLabel = "";
  export let color = "primary";
  export let outlined = false;
  export let placeholder = "";
  export let hint = "";
  export let error = false;
  export let append = "";
  export let dense = false;
  export let persistentHint = false;
  export let autocomplete = false;
  export let noUnderline = false;
  export let showList = false;
  export let classes = classesDefault;
  export let optionsClasses = optionsClassesDefault;

  export let inputWrapperClasses = noop;
  export let appendClasses = noop;
  export let labelClasses = noop;
  export let inputClasses = noop;
  export let prependClasses = noop;
  export let listClasses = noop;
  export let selectedClasses = noop;
  export let itemClasses = noop;

  export let add = "";
  export let remove = "";
  export let replace = "";

  let className = "";
  export {className as class};

  let filteredItems = items;
  let itemsProcessed = [];


  function process(it) {
    return it.map(i => typeof i !== "object"
     ? ({ value: i, text: i })
     : i);
  }

  $: itemsProcessed = process(items);

  onMount(() => {
    selectedLabel = getLabel(value);
  })

  const dispatch = createEventDispatcher();

  function getLabel(value) {
    return value !== undefined ? (itemsProcessed.find(i => i.value === value) || { text: "" }).text : "";
  }

  function filterItems({ target }) {
    filteredItems = itemsProcessed.filter(i =>
      i.text.toLowerCase().includes(target.value.toLowerCase())
    );
  }

  function handleInputClick() {
    if (autocomplete) {
      showList = true;
    } else {
      showList = !showList;
    }
  }

  const onHideListPanel = () => showList = false;

  const cb = new ClassBuilder(classes, classesDefault);
  $: c = cb
    .flush()
    .add(classes, true, classesDefault)
    .add(className)
    .get();

  const ocb = new ClassBuilder(optionsClasses, optionsClassesDefault);
  $: o = ocb
    .flush()
    .add(optionsClasses, true, optionsClassesDefault)
    .add("rounded-t-none", !outlined)
    .get();
</script>

<div class={c} use:hideListAction={onHideListPanel}>
  <slot name="select">
    <TextField
      select
      {dense}
      focused={showList}
      {autocomplete}
      value={selectedLabel}
      {outlined}
      {label}
      {placeholder}
      {hint}
      {error}
      {append}
      {persistentHint}
      {color}
      {add}
      {remove}
      {replace}
      {noUnderline}
      class={inputWrapperClasses}
      {appendClasses}
      {labelClasses}
      {inputClasses}
      {prependClasses}
      on:click={handleInputClick}
      on:click-append={(e => showList = !showList)}
      on:click
      on:input={filterItems}
      append="arrow_drop_down"
      appendReverse={showList}
    />
  </slot>

  {#if showList}
    <slot name="options">
      <div
        class={o}
        on:click={() => (showList = false)}
      >
        <List
          bind:value
          class={listClasses}
          {selectedClasses}
          {itemClasses}
          select
          {dense}
          items={filteredItems}
          on:change={({ detail }) => {
            selectedLabel = getLabel(detail);
            dispatch('change', detail);
          }} />
      </div>
    </slot>
  {/if}
</div>
