<script>
  import { getContext , onDestroy} from "svelte";
  import CellLink from "../../bb_super_components_shared/src/lib/SuperCell/cells/CellLink.svelte";

  const { styleable, Block, BlockComponent, Provider } = getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const formApi = formContext?.formApi;

  export let field;
  export let validation

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

  export let label;
  export let span = 6;
  export let placeholder
  export let defaultValue
  export let disabled
  export let readonly

  export let icon

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
    validation,
    formStep
  )

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
    fieldSchema = value?.fieldSchema;
  });

  $: value = fieldState?.value ? fieldState.value : []
  $: error = fieldState?.error


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
  
  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
  })

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<Block>
  <div
    class="superField"
    use:styleable={$component.styles}  
  >
    {#if label || error }
      <label for="superCell" class="superlabel" class:bound={formContext}>
        {label}
      </label>
    {/if}
    
    <div class="inline-cells">
      <CellLink
        bind:cellState
        cellOptions={{ 
          placeholder, 
          readonly,
          disabled,
          defaultValue,
          controlType,
          role: "formInput", 
          icon: icon,
          }}
        {value}
        {fieldSchema}
        on:change={(e) => fieldApi?.setValue(e.detail)}
        on:blur={cellState.lostFocus}
      />

      {#if customButtons && buttons?.length}
        <div
          class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
        >
        <Provider data={ { value : fieldState.value }} >
          {#each buttons as { text, onClick , quiet, disabled, type }}
            <BlockComponent
              type = "plugin/bb-component-SuperButton"
              props = {{
                quiet,
                disabled, 
                size: "M",
                text,
                onClick,
                emphasized : true,
                selected: type == "cta"
              }}>
              </BlockComponent>
            {/each}
        </Provider>
      </div>
    {/if}
    </div>
  </div>
</Block>

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
  .superlabel {
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

  .disabled {
    color: var(--spectrum-global-color-gray-600);
    background-color: var(--spectrum-global-color-gray-200);
  }

  .superlabel.bound {
    gap: 0.5rem;
  }
</style>




