<script>
  import { getContext, onDestroy } from "svelte";
  import {
    CellLink,
    SuperButton,
    SuperField,
  } from "@poirazis/supercomponents-shared";

  const { API, styleable, builderStore, fetchData, enrichButtonActions } =
    getContext("sdk");

  const component = getContext("component");
  const allContext = getContext("context");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const groupLabelPosition = getContext("field-group");
  const groupColumns = getContext("field-group-columns");
  const labelWidth = getContext("field-group-label-width");
  const groupDisabled = getContext("field-group-disabled");
  const formApi = formContext?.formApi;

  export let field;
  export let filter;
  export let validation;

  export let controlType = "select";
  export let wide;
  export let buttons = [];

  export let label;
  export let span = 6;
  export let placeholder = "Choose an Option";
  export let defaultValue;
  export let disabled;
  export let readonly;
  export let search;
  export let helpText;

  export let tableId;

  export let icon;

  export let onChange;

  export let relViewMode = "text";
  export let role = "formInput";
  export let labelPosition = "fieldGroup";
  export let showDirty = false;

  export let ownId;

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let fieldSchema;
  let value;
  let fetch;
  let enrichedDefaultValue;
  let has_self_ref_relationship;

  $: multirow = controlType != "select";
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: labelPos =
    groupLabelPosition && labelPosition == "fieldGroup"
      ? groupLabelPosition
      : labelPosition;
  $: value = fieldState?.value;
  $: error = fieldState?.error;

  $: formField = formApi?.registerField(
    field,
    "link",
    defaultValue,
    disabled,
    readonly,
    validation,
    formStep
  );
  $: tableId = formContext?.dataSource?.tableId;
  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
    fieldSchema = value?.fieldSchema;
    value = fieldState?.value;
  });

  $: enrichDefaultValue(defaultValue);
  $: grabEnrichedValue($fetch?.rows);
  $: identifySelfRelationship(fieldSchema);
  $: is_self_relationship = field?.startsWith("fk_self_") ? field : undefined;
  $: if (is_self_relationship) {
    tableId = formContext?.dataSource?.tableId;
  }

  $: if ($builderStore.inBuilder && $component.selected) {
    builderStore.actions.updateProp("isSelfReferencing", is_self_relationship);
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "grid-column": span < 7 ? "span " + span : "span " + groupColumns * 6,
    },
  };

  const handleChange = (e) => {
    if (is_self_relationship) {
      let val = e.detail;
      if (val && val.length) {
        let id = val[0]._id;
        fieldApi?.setValue(id);
      }
    } else {
      fieldApi?.setValue(e.detail);
      onChange?.({ value: fieldState.value });
    }
  };

  const enrichDefaultValue = (val) => {
    if (val) {
      fetchRow(fieldSchema?.tableId, val);
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
              id: rowId,
            },
          },
          paginate: false,
          limit: 1,
        },
      });
  };

  const identifySelfRelationship = async (fs) => {
    if (!fs || !fs?.tableId) return;

    await API.fetchTableDefinition(fs.tableId).then((res) => {
      let fields = Object.keys(res?.schema);
      has_self_ref_relationship = fields.find((x) => x.startsWith("fk_self_"));
    });
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<div use:styleable={$component.styles}>
  <SuperField
    {labelPos}
    {labelWidth}
    {field}
    {label}
    {error}
    {helpText}
    {multirow}
  >
    {#key enrichedDefaultValue}
      <CellLink
        cellOptions={{
          placeholder,
          search,
          readonly: readonly || fieldState?.readonly,
          disabled: disabled || groupDisabled || fieldState?.disabled,
          controlType,
          error: fieldState?.error,
          role,
          icon,
          showDirty,
          joinColumn: has_self_ref_relationship ?? is_self_relationship,
          relViewMode,
          wide,
        }}
        {value}
        {ownId}
        fieldSchema={{
          ...fieldSchema,
          tableId: is_self_relationship
            ? formContext?.dataSource?.tableId
            : fieldSchema?.tableId,
          relationshipType: is_self_relationship
            ? "self"
            : fieldSchema?.relationshipType,
          recursiveTable: is_self_relationship || has_self_ref_relationship,
        }}
        {filter}
        on:change={handleChange}
      />

      {#if buttons?.length && !fieldState.disabled}
        <div class="inline-buttons">
          {#each buttons as { icon, text, onClick, quiet, disabled, type, size }}
            <SuperButton
              {icon}
              {quiet}
              {disabled}
              {size}
              {type}
              {text}
              on:click={enrichButtonActions(onClick, $allContext)({ value })}
            />
          {/each}
        </div>
      {/if}
    {/key}
  </SuperField>
</div>
