<script>
  import { getContext , onDestroy} from "svelte";
  import { SuperCell } from "../../bb_super_components_shared/src/lib"

  const { styleable, builderStore, componentStore } = getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const formApi = formContext?.formApi;

  export let field;
  export let controlType
  
  export let customButtons

  export let frontButtons = [];
  export let frontButtonsQuiet;
  export let frontButtonsGrouping;
  export let frontButtonsSelected = [0];
  export let buttons = [];
  export let buttonsQuiet;
  export let buttonsGrouping;
  export let buttonsSelected = [0];

  export let fieldLabel;
  export let span = 6;
  export let inForm = false;
  export let placeholder
  export let defaultValue
  export let disabled
  export let readonly

  export let frontIcon

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema
  let value;
  let cellState
  

  $: formStep = formStepContext ? $formStepContext || 1 : 1;

  $: formField = formApi?.registerField(
    field,
    "link",
    defaultValue,
    disabled,
    readonly,
    null,
    formStep
  )

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
    fieldSchema = value?.fieldSchema;
  });

  $: value = fieldState?.value ? fieldState.value : defaultValue

  $: {
    if (
      $builderStore.inBuilder &&
      $componentStore.selectedComponentPath?.includes($component.id) &&
      formContext &&
      !inForm
    ) {
      builderStore.actions.updateProp("inForm", true);
    } else if (
      $builderStore.inBuilder &&
      $componentStore.selectedComponentPath?.includes($component.id) &&
      inForm &&
      !formContext
    ) {
      builderStore.actions.updateProp("inForm", false);
    }
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "flex-direction": labelPos == "left" ? "row" : "column",
      gap: labelPos == "left" ? "0.85rem" : "0rem",
      "grid-column": labelPos ? "span " + span : null,
      "--label-width":
        labelPos == "left" ? (labelWidth ? labelWidth : "6rem") : "auto",
    },
  };
  
  const buttonClick = (group, idx) => {
    if (group == "front" && frontButtonsGrouping == "single") {
      frontButtonsSelected = [idx];
    } else if (group == "end" && buttonsGrouping == "single") {
      buttonsSelected = [idx];
    }

    if (group == "front" && frontButtonsGrouping == "multi") {
      let i = frontButtonsSelected.indexOf(idx);
      if (i > -1) frontButtonsSelected.splice(i, 1);
      else frontButtonsSelected.push(idx);

      frontButtonsSelected = frontButtonsSelected;
    } else if (group == "end" && buttonsGrouping == "multi") {
      buttonsSelected = [idx];
    }
  };

  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
  })

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div
  class="superField"
  on:focus={cellState.focus} 
  tabindex="0"
  use:styleable={$component.styles}  
>
  <label for="superCell" class="superFieldLabel" class:bound={formContext}>
    {fieldLabel || field || "Unamed Field"}
  </label>
  
  <div class="inline-cells">
    {#if customButtons && frontButtons?.length}
      <div
        class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
        class:spectrum-ActionGroup--quiet={frontButtonsQuiet}
      >
        {#each frontButtons as button, idx}
          <button
            on:click={(e) => buttonClick("front", idx)}
            class="spectrum-ActionButton spectrum-ActionButton--sizeM"
            class:spectrum-ActionButton--quiet={frontButtonsQuiet ||
              button.quiet}
            class:warning={button.type == "warning"}
            class:cta={button.type == "cta"}
            class:disabled={button.disabled}
            class:is-selected={frontButtonsGrouping &&
              frontButtonsSelected.includes(idx)}
          >
            <span class="spectrum-ActionButton-label">{@html button?.text}</span
            >
          </button>
        {/each}
      </div>
    {/if}

    <SuperCell
      bind:cellState
      cellOptions={{ 
        placeholder, 
        defaultValue,
        controlType,
        role: "formInput", 
        iconFront: frontIcon,
        }}
      {value}
      {fieldSchema}
      editable
      on:change={(e) => fieldApi?.setValue(e.detail)}
      on:blur={cellState.lostFocus}
    />

    {#if customButtons && buttons?.length}
      <div
        class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
        class:spectrum-ActionGroup--quiet={buttonsQuiet}
      >
        {#each buttons as button, idx}
          <button
            class="spectrum-ActionButton spectrum-ActionButton--sizeM"
            on:click={(e) => buttonClick("end", idx)}
            class:spectrum-ActionButton--quiet={buttonsQuiet || button.quiet}
            class:warning={button.type == "warning"}
            class:cta={button.type == "cta"}
            class:disabled={button.disabled}
            class:is-selected={buttonsGrouping && buttonsSelected.includes(idx)}
          >
            <span class="spectrum-ActionButton-label">{button.text}</span>
          </button>
        {/each}
      </div>
    {/if}
  </div>
</div>

<style>
  .superField {
    display: flex;
    align-items: stretch;
    justify-content: stretch;
    min-width: 0;
  }

  .superField:focus {
    outline: none;
  }
  .superFieldLabel {
    display: flex;
    align-items: flex-start;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    min-width: var(--label-width);
    max-width: var(--label-width);
    font-size: 12px;
    line-height: 1.75rem;
    font-weight: 400;
    color: var(--spectrum-global-color-gray-700);
  }

  .inline-cells {
    flex: 1;
    display: flex;
    justify-items: stretch;
    height: 2rem;
  }
  .warning {
    color: var(--spectrum-global-color-red-500);
    border-color: var(--spectrum-global-color-red-500);
  }

  .spectrum-ActionButton--quiet.warning {
    color: var(--spectrum-global-color-red-500);
    border: none;
  }

  .warning:hover {
    color: #000;
    background-color: var(--spectrum-global-color-red-500);
  }
  .cta {
    color: #fff;
    background-color: var(--primaryColor);
  }

  .disabled {
    color: var(--spectrum-global-color-gray-600);
    background-color: var(--spectrum-global-color-gray-200);
  }

  .superFieldLabel.bound {
    gap: 0.5rem;
  }
</style>




