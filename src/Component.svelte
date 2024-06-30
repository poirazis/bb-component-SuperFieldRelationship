<script>
  import { getContext, onDestroy } from "svelte";
  import CellLink from "../../bb_super_components_shared/src/lib/SuperTableCells/CellLink.svelte";
  import "../../bb_super_components_shared/src/lib/SuperFieldsCommon.css";

  const {
    styleable,
    Block,
    BlockComponent,
    Provider,
    builderStore,
    componentStore,
  } = getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const groupDisabled = getContext("field-group-disabled");
  const formApi = formContext?.formApi;

  export let field;
  export let filter;
  export let validation;

  export let controlType = "select";
  export let customButtons;
  export let buttons = [];
  export let buttonsQuiet;

  export let label;
  export let span = 6;
  export let placeholder;
  export let defaultValue;
  export let disabled;
  export let readonly;
  export let autocomplete;

  export let tableId;
  export let labelColumn;
  export let valueColumn;
  export let parentColumn;

  export let pickerColumns = [];
  export let searchColumns;

  export let multi;
  export let icon;

  export let onChange;

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema;
  let value;
  let innerLabelColumn = labelColumn;

  $: formStep = formStepContext ? $formStepContext || 1 : 1;

  $: formField = formApi?.registerField(
    field,
    "link",
    defaultValue,
    disabled,
    readonly,
    validation,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
    fieldSchema = value?.fieldSchema;
  });

  $: value = fieldState?.value ? fieldState.value : [];

  $: if (
    $builderStore.inBuilder &&
    fieldSchema?.tableId &&
    tableId?.tableId != fieldSchema?.tableId &&
    $componentStore.selectedComponentPath?.includes($component.id)
  ) {
    builderStore.actions.updateProp("tableId", {
      tableId: fieldSchema?.tableId,
      type: "table",
    });
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "flex-direction": labelPos == "left" ? "row" : "column",
      "align-items": "stretch",
      gap: labelPos == "left" ? "0.5rem" : "0rem",
      "grid-column": labelPos ? "span " + span : "span 1",
      "--label-width":
        labelPos == "left" ? (labelWidth ? labelWidth : "6rem") : "auto",
    },
  };

  const handleChange = (e) => {
    fieldApi?.setValue(e.detail);
    onChange?.({ value: fieldState.value });
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<Block>
  <div class="superField" use:styleable={$component.styles}>
    {#if label}
      <label for="superCell" class="superlabel" class:left={labelPos == "left"}>
        {label}
        {#if fieldState.error}
          <div class="error" class:left={labelPos == "left"}>
            <span>{fieldState.error}</span>
          </div>
        {/if}
      </label>
    {/if}

    <div class="inline-cells">
      <CellLink
        cellOptions={{
          placeholder,
          autocomplete,
          readonly: readonly || fieldState?.readonly,
          disabled: disabled || groupDisabled || fieldState?.disabled,
          defaultValue,
          controlType,
          padding: "0.5rem",
          role: "formInput",
          icon: icon,
          multi,
          pickerColumns,
          searchColumns,
        }}
        {value}
        {fieldSchema}
        tableId={tableId?.tableId}
        {valueColumn}
        labelColumn={innerLabelColumn}
        {parentColumn}
        {filter}
        multi={fieldSchema.relationshipType != "one-to-many"}
        on:change={handleChange}
      />

      {#if customButtons && buttons?.length}
        <div
          class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
        >
          <Provider data={{ value: fieldState.value }}>
            {#each buttons as { text, onClick, quiet, disabled, type }}
              <BlockComponent
                type="plugin/bb-component-SuperButton"
                props={{
                  quiet: buttonsQuiet,
                  disabled,
                  size: "M",
                  text,
                  onClick,
                  emphasized: true,
                  selected: type == "cta",
                }}
              ></BlockComponent>
            {/each}
          </Provider>
        </div>
      {/if}
    </div>
  </div>
</Block>
