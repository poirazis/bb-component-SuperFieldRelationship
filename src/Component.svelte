<script>
  import { getContext, onDestroy } from "svelte";
  import CellLink from "../../bb_super_components_shared/src/lib/SuperTableCells/CellLink.svelte";
  import SuperList from "../../bb_super_components_shared/src/lib/SuperList/SuperList.svelte";
  import "../../bb_super_components_shared/src/lib/SuperFieldsCommon.css";

  const {
    API,
    styleable,
    Block,
    BlockComponent,
    Provider,
    builderStore,
    fetchData,
  } = getContext("sdk");

  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const groupLabelPosition = getContext("field-group");
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
  export let search;
  export let clearValueIcon;

  export let tableId;

  export let pickerColumns = [];
  export let searchColumns;

  export let multi;
  export let icon;

  export let onChange;
  export let onItemClick;

  export let relViewMode = "text";
  export let role = "formInput";
  export let labelPosition;

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema;
  let value;
  let fetch;
  let enrichedDefaultValue;
  let has_self_ref_relationship;

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: labelPos = labelPosition ? labelPosition : groupLabelPosition || "left";
  $: value = fieldState.value;

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

  $: enrichDefaultValue(defaultValue);
  $: grabEnrichedValue($fetch?.rows);
  $: identifySelfRelationship(fieldSchema);

  $: if (
    $builderStore.inBuilder &&
    $component.selected &&
    fieldSchema?.tableId &&
    tableId?.tableId != fieldSchema?.tableId
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

  const enrichDefaultValue = (val) => {
    if (val) {
      // Is the defaultValue a row id ?
      if (val.startsWith("ro_ta_")) {
        fetchRow(fieldSchema?.tableId, val);
      }
      // Will attemp to match with primaryDisplay
      else {
      }
    }
  };

  const grabEnrichedValue = (rows) => {
    if (rows?.length && !enrichedDefaultValue) {
      enrichedDefaultValue = [
        {
          _id: rows[0]["_id"],
          primaryDisplay: rows[0][$fetch.definition.primaryDisplay],
        },
      ];
      fieldApi.setValue(enrichedDefaultValue);
      value = enrichedDefaultValue;
    }
  };

  const fetchRow = (tableId, rowId) => {
    if (tableId && rowId)
      fetch = fetchData({
        API,
        datasource: {
          tableId,
          type: "table",
        },
        options: {
          query: {
            equal: {
              _id: rowId,
            },
          },
          paginate: false,
          limit: 1,
        },
      });
  };

  const identifySelfRelationship = async (fs) => {
    if (!fs || !fs.tableId) return;

    await API.fetchTableDefinition(fs.tableId).then((res) => {
      let fields = Object.keys(res?.schema);
      has_self_ref_relationship =
        fields.find((x) => x.endsWith("_self_")) || undefined;
    });
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<Block>
  <div
    class="superField"
    class:multirow={controlType == "expanded"}
    use:styleable={$component.styles}
  >
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
    {#key enrichedDefaultValue}
      <div class="inline-cells">
        <CellLink
          cellOptions={{
            placeholder,
            search,
            readonly: readonly || fieldState?.readonly,
            disabled: disabled || groupDisabled || fieldState?.disabled,
            controlType,
            error: fieldState?.error,
            padding: "0.5rem",
            role,
            icon: icon,
            clearValueIcon,
            joinColumn: has_self_ref_relationship,
            multi,
            pickerColumns,
            searchColumns,
            relViewMode,
            onItemClick,
          }}
          {value}
          {fieldSchema}
          {filter}
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
                    quiet: buttonsQuiet || quiet,
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
