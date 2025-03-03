<script lang="ts">
// https://v3.vuejs.org/api/sfc-script-setup.html#usage-alongside-normal-script
export default {
  inheritAttrs: false
};
</script>

<script setup lang="ts">
import { omit } from 'lodash';
import { computed, onMounted, ref, useAttrs } from 'vue';

import { HtmlInputEvent } from '@/types';

import useInputEvents from './composables/useInputEvents';
import useInputStyles from './composables/useInputStyles';
import useInputValidation from './composables/useInputValidation';

/**
 * TYPES
 */
type InputValue = string | number;
type InputType = 'text' | 'number' | 'date' | 'email' | 'password';
type InputSize = 'xs' | 'sm' | 'md' | 'lg';
type ValidationTrigger = 'input' | 'blur';
type RuleFunction = (val: InputValue) => string;
export type Rules = RuleFunction[];

type Props = {
  name: string;
  modelValue: InputValue;
  isValid?: boolean;
  type?: InputType;
  size?: InputSize;
  disabled?: boolean;
  label?: string;
  inputAlignRight?: boolean;
  decimalLimit?: number;
  validateOn?: ValidationTrigger;
  rules?: Rules;
  noRadius?: boolean;
  noShadow?: boolean;
  noBorder?: boolean;
  autoFocus?: boolean;
  format?: (input: string | number) => string | number;
};

/**
 * PROPS & EMITS
 */
const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  modelValue: '',
  isValid: true,
  size: 'md',
  disabled: false,
  inputAlignRight: false,
  decimalLimit: 18,
  validateOn: 'blur',
  rules: () => [],
  noRadius: false,
  noShadow: false,
  noBorder: false,
  autoFocus: false
});

const emit = defineEmits<{
  (e: 'blur', value: string): void;
  (e: 'input', value: string): void;
  (e: 'update:modelValue', value: string): void;
  (e: 'update:isValid', value: boolean): void;
  (e: 'keydown', value: HtmlInputEvent);
  (e: 'mouseOver', value: Event);
  (e: 'mouseLeave', value: Event);
}>();

/**
 * STATE
 */
const textInput = ref<HTMLInputElement>();

/**
 * COMPOSABLES
 */
const attrs = useAttrs();
const { errors, isInvalid, validate } = useInputValidation(props, emit);
const {
  isActive,
  isHover,
  onInput,
  onKeydown,
  onBlur,
  onClick,
  onFocus,
  onMouseOver,
  onMouseLeave
} = useInputEvents(props, emit, validate);
const {
  parentClasses,
  inputContainerClasses,
  inputGroupClasses,
  headerClasses,
  footerClasses,
  inputClasses,
  prependClasses,
  appendClasses,
  borderRadiusClasses
} = useInputStyles(props, isInvalid, isActive, isHover, attrs);

/**
 * COMPUTED
 */
// We don't want to pass on parent level classes to the html
// input element. So we need to remove it from the attrs object.
const inputAttrs = computed(() => omit(attrs, 'class'));

/**
 * LIFECYCLE
 */
onMounted(() => {
  textInput.value?.focus();
});
</script>

<template>
  <div :class="['bal-text-input', parentClasses, borderRadiusClasses]">
    <div
      :class="['input-container', inputContainerClasses, borderRadiusClasses]"
      @mouseover="onMouseOver"
      @mouseleave="onMouseLeave"
    >
      <div v-if="$slots.header || label" :class="['header', headerClasses]">
        <slot name="header">
          <span class="label">
            {{ label }}
          </span>
        </slot>
      </div>
      <div :class="['input-group', inputGroupClasses]">
        <div v-if="$slots.prepend" :class="['prepend', prependClasses]">
          <slot name="prepend" />
        </div>
        <input
          ref="textInput"
          :type="type"
          :name="name"
          :value="modelValue"
          v-bind="inputAttrs"
          :disabled="disabled"
          @blur="onBlur"
          @input="onInput"
          @keydown="onKeydown"
          @click="onClick"
          @focus="onFocus"
          :class="['input', inputClasses]"
        />
        <div v-if="$slots.append" :class="['append', appendClasses]">
          <slot name="append" />
        </div>
      </div>
      <div v-if="$slots.footer" :class="['footer', footerClasses]">
        <slot name="footer" />
      </div>
      <div v-if="isInvalid && !!errors[0]" :class="['error']">
        {{ errors[0] }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.input-container {
  @apply bg-white dark:bg-gray-800 border transition-colors;
}

.input-group {
  @apply flex;
}

.input {
  @apply flex-grow bg-transparent overflow-hidden;
}

.label {
  @apply text-sm text-gray-500;
}

.error {
  @apply text-xs text-red-500 mt-1 ml-1;
}
</style>
