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
    API,
  } = getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const formApi = formContext?.formApi;

  export let linkValueType = "link";
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
  export let filterCustom;
  export let limitCustom;
  export let valueColumn;
  export let labelColumn;

  export let pickerColumns = [];
  export let searchColumns;

  export let multi;
  export let icon;

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema;
  let value;
  let cellState;
  let definition;
  let loaded;
  let innerLabelColumn = labelColumn;
  let innerValueColumn;
  let innerPickerColumns;

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
  $: error = fieldState?.error;
  $: linkValueType == "link" && fieldSchema.tableId
    ? fetchDefinition(fieldSchema?.tableId)
    : null;

  $: if (linkValueType == "link" && definition) {
    innerValueColumn = "_id";
    innerLabelColumn = definition.primaryDisplay;
    innerPickerColumns =
      pickerColumns?.length > 0 ? pickerColumns : [{ name: innerLabelColumn }];
  }

  $: innerLimit = limitCustom ? limitCustom : 10;
  $: if (
    $builderStore.inBuilder &&
    linkValueType == "link" &&
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
      gap: labelPos == "left" ? "0.85rem" : "0rem",
      "grid-column": labelPos ? "span " + span : null,
      "--label-width":
        labelPos == "left" ? (labelWidth ? labelWidth : "6rem") : "auto",
    },
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const fetchDefinition = async (tableId) => {
    try {
      definition = await API.fetchTableDefinition(tableId);
    } catch (error) {
      definition = null;
    }

    if (!loaded) {
      loaded = true;
    }
  };
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

    {#key loaded}
      <div class="inline-cells">
        <CellLink
          bind:cellState
          cellOptions={{
            placeholder,
            autocomplete,
            readonly,
            disabled,
            defaultValue,
            controlType,
            role: "formInput",
            icon: icon,
            multi,
            pickerColumns,
            searchColumns,
          }}
          {value}
          {fieldSchema}
          tableId={tableId?.tableId}
          valueColumn={innerValueColumn}
          labelColumn={innerLabelColumn}
          {filter}
          ,
          multi={linkValueType == "array" ||
            (linkValueType == "link" &&
              fieldSchema.relationshipType != "one-to-many")}
          {innerLimit}
          pickerColumns={innerPickerColumns ?? pickerColumns}
          on:change={(e) => fieldApi?.setValue(e.detail)}
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
                    quiet,
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
    {/key}
  </div>
</Block>
