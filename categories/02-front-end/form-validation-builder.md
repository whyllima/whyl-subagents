---
name: form-validation-builder
description: Creates validated Vue 3 forms using vee-validate 4 + Yup — single forms, multi-step wizards, reusable field widgets, and nested validation schemas.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Form Validation Builder

Creates: Vue 3 forms with vee-validate 4 and Yup validation — from simple single-page forms to complex multi-step wizards with reusable field components.

## Core Architecture

```
<Form>                          ← vee-validate Form (provides form context)
  ├── :validation-schema        ← Yup schema (defines ALL validation rules)
  ├── :initial-values           ← default field values (reactive)
  └── <ChildComponents />       ← use useField(name) to bind to form context
```

**One rule:** any component inside `<Form>` that calls `useField('fieldName')` automatically registers with that form. No prop drilling, no provide/inject — it just works.

## 1. Simple Form

```vue
<script setup lang="ts">
import { Form } from 'vee-validate';
import * as yup from 'yup';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();

const schema = yup.object().shape({
    name: yup.string().min(3, t('validation.minLength', { min: 3 })).required(t('validation.required')),
    email: yup.string().email(t('validation.email')).required(t('validation.required')),
    status: yup.string().required(t('validation.required'))
});

const initialValues = { name: '', email: '', status: 'active' };

const onSubmit = (values: any) => {
    console.log('Valid form data:', values);
};
</script>

<template>
    <Form :validation-schema="schema" :initial-values="initialValues" @submit="onSubmit" v-slot="{ errors, isSubmitting }">
        <TextInput name="name" :label="t('field.name')" placeholder="John Doe" />
        <TextInput name="email" type="email" :label="t('field.email')" placeholder="john@example.com" />
        <DropdownInput name="status" :label="t('field.status')" :options="statusOptions" />
        <Button type="submit" label="Save" :loading="isSubmitting" />
    </Form>
</template>
```

## 2. Reusable Field Widget Pattern

Every field widget follows this structure — it calls `useField(name)` and renders the input + error message:

```vue
<!-- TextInput.vue -->
<script setup lang="ts">
import { useField } from 'vee-validate';
import { toRef } from 'vue';

const props = defineProps({
    name: { type: String, required: true },
    label: { type: String, required: true },
    type: { type: String, default: 'text' },
    placeholder: { type: String, default: '' },
    disabled: { type: Boolean, default: false },
    maxLength: { type: String, default: '' }
});

const name = toRef(props, 'name');
const { value: inputValue, errorMessage, handleBlur, meta } = useField<string>(name.value);
</script>

<template>
    <div class="field-wrapper" :class="{ 'has-error': !!errorMessage }">
        <label :for="name" class="field-label">{{ label }}</label>
        <InputText
            :id="name"
            :type="type"
            :placeholder="placeholder"
            :maxLength="maxLength"
            :disabled="disabled"
            v-model="inputValue"
            :class="{ 'p-invalid': errorMessage }"
            @blur="handleBlur"
        />
        <small v-if="errorMessage" class="p-error">{{ errorMessage }}</small>
    </div>
</template>
```

**Other widget examples:**

```vue
<!-- CalendarInput.vue -->
<script setup lang="ts">
import { useField } from 'vee-validate';
import { toRef } from 'vue';

const props = defineProps({
    name: { type: String, required: true },
    label: { type: String, required: true },
    minDate: { type: Date, default: null },
    maxDate: { type: Date, default: null },
    disabled: { type: Boolean, default: false }
});

const name = toRef(props, 'name');
const { value: inputValue, errorMessage, meta } = useField<Date>(name.value);
</script>

<template>
    <div class="field-wrapper" :class="{ 'has-error': !!errorMessage }">
        <label :for="name" class="field-label">{{ label }}</label>
        <Calendar :id="name" v-model="inputValue" :minDate="minDate" :maxDate="maxDate"
                  :disabled="disabled" :class="{ 'p-invalid': errorMessage }" dateFormat="dd/mm/yy" showIcon />
        <small v-if="errorMessage" class="p-error">{{ errorMessage }}</small>
    </div>
</template>
```

```vue
<!-- DropdownInput.vue -->
<script setup lang="ts">
import { useField } from 'vee-validate';
import { toRef } from 'vue';

const props = defineProps({
    name: { type: String, required: true },
    label: { type: String, required: true },
    options: { type: Array, required: true },
    optionLabel: { type: String, default: 'name' },
    optionValue: { type: String, default: 'value' },
    placeholder: { type: String, default: '' },
    disabled: { type: Boolean, default: false }
});

const name = toRef(props, 'name');
const { value: inputValue, errorMessage, handleBlur } = useField(name.value);
</script>

<template>
    <div class="field-wrapper" :class="{ 'has-error': !!errorMessage }">
        <label :for="name" class="field-label">{{ label }}</label>
        <Dropdown :id="name" v-model="inputValue" :options="options" :optionLabel="optionLabel"
                  :optionValue="optionValue" :placeholder="placeholder" :disabled="disabled"
                  :class="{ 'p-invalid': errorMessage }" @blur="handleBlur" />
        <small v-if="errorMessage" class="p-error">{{ errorMessage }}</small>
    </div>
</template>
```

## 3. Multi-Step Wizard Pattern

The key insight: **one `<Form>` wraps the entire wizard**. Each step is a child component that uses `useField` to register its fields with the parent form. Steps validate their fields before advancing.

### 3a. Page Wrapper (owns the Form)

```vue
<script setup lang="ts">
import { ref, computed } from 'vue';
import { Form } from 'vee-validate';
import * as yup from 'yup';

const currentStep = ref(1);

const schema = yup.object().shape({
    // Step 1 fields
    name: yup.string().min(3).required('Name is required'),
    start_date: yup.date().required('Start date is required').typeError('Start date is required'),
    end_date: yup.date().required('End date is required').typeError('End date is required'),
    // Step 2 fields
    category: yup.string().required('Category is required'),
    description: yup.string().max(500).optional(),
    // Step N fields...
});

const initialValues = computed(() => ({
    name: '', start_date: null, end_date: null,
    category: '', description: ''
}));
</script>

<template>
    <Form :validation-schema="schema" :initial-values="initialValues" v-slot="{ errors }">
        <StepOne v-if="currentStep === 1" @next="currentStep = 2" />
        <StepTwo v-else-if="currentStep === 2" @next="currentStep = 3" @previous="currentStep = 1" />
        <StepThree v-else @previous="currentStep = 2" @complete="handleSubmit" />
    </Form>
</template>
```

### 3b. Step Component (validates before advancing)

```vue
<!-- StepOne.vue -->
<script setup lang="ts">
import { useField } from 'vee-validate';

const emit = defineEmits(['next']);

// Access fields for validation — widgets own the v-model
const { validate: validateName } = useField<string>('name');
const { value: startDateValue, validate: validateStartDate } = useField<Date | null>('start_date');
const { validate: validateEndDate } = useField<Date | null>('end_date');

const handleNext = async () => {
    // Validate ALL fields in this step before advancing
    const [nameResult, startResult, endResult] = await Promise.all([
        validateName(),
        validateStartDate(),
        validateEndDate()
    ]);

    if (!nameResult.valid || !startResult.valid || !endResult.valid) return;
    emit('next');
};
</script>

<template>
    <div>
        <!-- Widgets handle useField internally — they show errors automatically -->
        <TextInput name="name" label="Name" placeholder="Enter name" />
        <CalendarInput name="start_date" label="Start Date" :minDate="new Date()" />
        <CalendarInput name="end_date" label="End Date" :minDate="startDateValue || new Date()" />
        <Button label="Next" @click="handleNext" />
    </div>
</template>
```

### 3c. Important: useField in widgets vs parent

```
<Form schema={yup}>                    ← Form provides context
  └── <StepOne>
        ├── useField('name')           ← for validate() access ONLY
        ├── <TextInput name="name">
        │     └── useField('name')     ← for v-model + error display
        └── Both share the SAME form field (by name)
```

When widget and parent both call `useField('name')`:
- They share the same value and validation state
- The widget renders the input + error message
- The parent calls `validate()` to trigger validation on demand
- This is the intended vee-validate pattern for reusable inputs

## 4. Yup Schema Patterns

### Basic types
```typescript
yup.string().required('Required')
yup.string().min(3, 'Min 3 characters').max(120, 'Max 120')
yup.string().email('Invalid email')
yup.number().min(0).max(100).required()
yup.date().required('Required').typeError('Must be a valid date')
yup.boolean()
yup.array().of(yup.string()).optional()
```

### Conditional validation
```typescript
yup.object().shape({
    type: yup.string().required(),
    // Only required when type is 'scheduled'
    schedule_date: yup.date().when('type', {
        is: 'scheduled',
        then: (schema) => schema.required('Schedule date required'),
        otherwise: (schema) => schema.nullable()
    })
});
```

### Nested objects
```typescript
yup.object().shape({
    user: yup.object().shape({
        name: yup.string().required(),
        email: yup.string().email().required()
    }),
    address: yup.object().shape({
        street: yup.string().required(),
        city: yup.string().required()
    })
});
```

### Arrays of objects
```typescript
yup.object().shape({
    entities: yup.array().of(
        yup.object().shape({
            name: yup.string().required(),
            type: yup.string().required(),
            variables: yup.array().of(
                yup.object().shape({
                    name: yup.string().required(),
                    value: yup.string().optional()
                })
            ).optional()
        })
    )
});
```

## 4b. Advanced Wizard Patterns

Real wizards need more than `.when()` — async checks, cross-step conditions, capability gating, and preserved values. These patterns cover all of it.

### Async validation (server-side uniqueness check)

Use `.test()` with an async function. The test runs after sync rules pass, so guard against empty values — `required`/`min` already flagged those:

```typescript
name: yup
    .string()
    .min(3, t('validation.minLength', { min: 3 }))
    .required(t('validation.required'))
    .test(
        'name-available',
        t('validation.nameTaken'),
        async (value) => {
            // Skip async check when sync rules already failed
            if (!value || value.trim().length < 3) return true;
            const res = await checkNameAvailability(value.trim(), currentUuid);
            return res.available;
        }
    )
```

**Tip:** debounce the API call in the function you import, not in yup — yup re-runs the schema on every keystroke.

### Accessing root/parent values from inside a `.test()`

Yup exposes the ancestor chain via `this.from`. Index `0` is the immediate parent shape, `1` is the next ancestor, and so on up to the root. Use this to gate validation by a flag defined in another step/branch:

```typescript
trigger: yup.object().shape({
    useExternalConditions: yup.boolean(),
    campaigns: yup
        .mixed()
        .test('campaigns-required', t('validation.campaignRequired'), function (val) {
            // `this.from[1].value` = the root form values
            const root = (this as any).from?.[1]?.value;
            if (!root?.capabilities?.trigger) return true;  // capability-gate: skip if feature off

            // `this.parent` = immediate parent (trigger object)
            const parent: any = this.parent;
            if (parent.multiGeo?.enabled) return true;       // conditional skip

            return Array.isArray(val) && val.length > 0;
        })
})
```

**The three scope handles inside `.test()`:**
| Handle | What it points to |
|---|---|
| `this.parent` | The object shape that contains this field |
| `this.from[0].value` | Same as `this.parent` |
| `this.from[1].value` | Grandparent (usually the root in 2-level wizards) |
| `this.from[2].value` | Great-grandparent (for deeply nested steps) |

### Capability-gated validation (feature flags)

Every test in a feature-gated step must first check the flag and early-return `true` when off. This keeps the schema in one place while letting the UI opt-in/out:

```typescript
const gatedTest = function (val: any) {
    const root = (this as any).from?.[1]?.value;
    if (!root?.capabilities?.trigger) return true;  // ← gate
    // ... real validation below
};
```

### Validating complex structures with `yup.mixed().test()`

When a field is a free-form array/object (grid, dynamic groups, list of variables), skip `yup.array().of(...)` and use `yup.mixed().test()` — you get full control:

```typescript
// 7x24 boolean grid — at least one cell must be true
daypartGrid: yup.mixed().test('has-selection', t('validation.selectAtLeastOne'), function (val) {
    const root = (this as any).from?.[1]?.value;
    if (!root?.capabilities?.trigger) return true;
    return Array.isArray(val) && val.some((row: any) => Array.isArray(row) && row.some(Boolean));
}),

// Array of groups — every group must have >= 1 condition, each condition fully filled
conditionGroups: yup.mixed().test('groups-valid', t('validation.conditionsRequired'), function (val) {
    const groups = val as any[];
    if (!Array.isArray(groups) || groups.length === 0) return false;
    return groups.every((g) =>
        Array.isArray(g.conditions) && g.conditions.length > 0 &&
        g.conditions.every((c: any) =>
            c.param && c.op && c.value !== '' && c.value !== null && c.value !== undefined
        )
    );
}),

// Array of variables — each with unique snake_case name and non-empty default
contextualVariables: yup.mixed().test('vars-valid', t('validation.varsInvalid'), function (val) {
    const vars = val as any[] | undefined;
    if (!Array.isArray(vars) || vars.length === 0) return false;
    const names = new Set<string>();
    for (const v of vars) {
        if (!v.name?.trim()) return false;
        if (!/^[a-z][a-z0-9_]*$/.test(v.name)) return false;    // format
        if (names.has(v.name)) return false;                     // uniqueness
        names.add(v.name);
        if (!v.defaultValue?.toString().trim()) return false;
    }
    return true;
})
```

### Preserving values across steps — `:keep-values` + `:key` + reactive initial-values

Wizards that remount (edit mode, hydration, step swap) lose state by default. Three props together fix it:

```vue
<Form
    :key="formKey"                         ← bump to force full reset (new entity, discard)
    :validation-schema="schema"
    :initial-values="initialFormValues"    ← reactive — updates hydrate the form
    :keep-values="true"                    ← don't wipe values when fields unmount (step nav)
    v-slot="{ errors, values }"
>
    <WizardStepContainer :formValues="values" :errors="errors" />
</Form>
```

```typescript
// Reactive initial-values: refills when hydration resolves (edit mode)
const initialFormValues = computed(() => ({
    name: hydrated.value?.name ?? '',
    start_date: hydrated.value?.start_date ?? null,
    capabilities: { trigger: hydrated.value?.trigger ?? false, dco: hydrated.value?.dco ?? false },
    trigger: {
        useExternalConditions: false,
        conditionGroups: [],
        groupOperator: 'AND',
        campaigns: [],
        triggerAction: null
    },
    creative: { contextualVariables: [], selectedStamps: [], regional: { enabled: false, values: [] } }
}));

// Force remount (e.g., user clicks "Start over")
const formKey = ref(0);
const resetWizard = () => formKey.value++;
```

**When to use what:**
| Need | Mechanism |
|---|---|
| User navigates between steps (fields unmount/remount) | `:keep-values="true"` |
| Hydrated data arrives asynchronously (edit mode) | `:initial-values` as `computed` |
| Full reset (discard draft, new entity) | Bump `:key="formKey"` |

### Wizard-level summary

A production wizard schema usually combines all five patterns: async `.test()` for uniqueness, `this.from[1].value` for root access, capability gating, `yup.mixed().test()` for complex structures, and `:keep-values` + reactive `:initial-values` on the Form. Keep the full schema in one file — do not split per step — so gating tests can reach across branches.

## 5. useField API Reference

```typescript
const {
    value,          // Ref<T> — reactive field value (v-model target)
    errorMessage,   // Ref<string> — current error message (empty if valid)
    errors,         // Ref<string[]> — all error messages
    meta,           // { valid, touched, dirty, pending }
    validate,       // () => Promise<{ valid: boolean; errors: string[] }>
    handleBlur,     // (event) => void — marks field as touched, triggers validation
    handleChange,   // (event) => void — updates value, triggers validation
    setValue,       // (value: T) => void — programmatically set value
    resetField,     // (state?) => void — reset to initial value
} = useField<T>('fieldName');
```

## 6. useForm API Reference (for form-level access)

```typescript
const {
    values,         // Reactive object with all field values
    errors,         // Reactive object with all field errors
    meta,           // { valid, touched, dirty, pending }
    validate,       // () => Promise<{ valid: boolean; errors: Record<string, string> }>
    setFieldValue,  // (field: string, value: any) => void
    setFieldError,  // (field: string, message: string) => void
    resetForm,      // (state?) => void
    handleSubmit,   // (onSuccess, onError?) => (event) => void
} = useForm({
    validationSchema: schema,
    initialValues: { ... }
});
```

## 7. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Widget and parent both render the same input | Widget renders input, parent only calls `validate()` |
| `useField` outside `<Form>` context | Creates orphan field. Ensure component is inside `<Form>` |
| Yup `.required()` on Date field doesn't work | Add `.typeError('message')` — null/undefined fails type check, not required |
| Validation only triggers on blur, not on button click | Call `validate()` explicitly in the click handler |
| Error message doesn't show | Ensure template has `<small v-if="errorMessage" class="p-error">{{ errorMessage }}</small>` |
| Field not resetting when form resets | Ensure `initialValues` is reactive (computed) and matches field names |
| Array fields not validating | Use `useFieldArray('entities')` for dynamic arrays |
| Async `.test()` firing on every keystroke | Debounce the API call inside the test function; guard early-return when sync rules haven't passed |
| `this.from[1].value` is `undefined` | You're at the root — use `this.parent` instead. `from[1]` only exists inside nested `yup.object().shape({...})` |
| Values wiped when navigating between wizard steps | Add `:keep-values="true"` on `<Form>` |
| Hydrated data not appearing in edit mode | Make `:initial-values` a `computed`, not a plain object |
| Validation skipped when capability flag toggles on mid-wizard | Every gated test must re-read `root.capabilities.X` via `this.from[1].value` — don't cache the flag |

## Workflow

1. Identify form type: simple form, modal form, or multi-step wizard
2. Read existing widgets: `Glob **/widgets/*Input*.vue` or `Glob **/widgets/*.vue`
3. Read existing forms for pattern reference: `Grep "Form.*validation-schema"` or `Grep "useField"`
4. Define Yup schema covering all fields
5. For wizards: wrap in single `<Form>`, each step validates its own fields
6. For widgets: create once, reuse everywhere — `useField(name)` inside widget
7. Verify: test validation triggers on blur AND on button click
