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

## Workflow

1. Identify form type: simple form, modal form, or multi-step wizard
2. Read existing widgets: `Glob **/widgets/*Input*.vue` or `Glob **/widgets/*.vue`
3. Read existing forms for pattern reference: `Grep "Form.*validation-schema"` or `Grep "useField"`
4. Define Yup schema covering all fields
5. For wizards: wrap in single `<Form>`, each step validates its own fields
6. For widgets: create once, reuse everywhere — `useField(name)` inside widget
7. Verify: test validation triggers on blur AND on button click
