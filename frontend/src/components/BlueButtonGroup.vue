<template>
  <div class="flex items-center">
    <div
      class="inline-flex overflow-hidden rounded-[6px] elevation-1"
      :class="[
        theme === 'dark' ? 'bg-[#464646]' : 'bg-[#00000011]',
        disabled ? 'opacity-50 pointer-events-none' : ''
      ]"
      :style="{ height: height || '30px' }"
    >
      <!-- Dynamic buttons + dividers -->
      <template
        v-for="(btn, idx) in buttonItems"
        :key="btn.name"
      >
        <button
          :disabled="disabled || btn.disabled"
          :style="{ backgroundColor: selected[idx] ? btn.activeColor || '#0B5087' : undefined }"
          class="flex items-center justify-center px-4 text-sm font-medium transition-colors duration-200"
          :class="[
            selected[idx]
              ? 'text-white'
              : theme === 'dark'
                ? 'text-[#ffffff99]'
                : 'text-[#00000066]',
            selected[idx] && type === 'switch' ? 'elevation-5' : 'elevation-0',
            (disabled || btn.disabled)
              ? 'cursor-not-allowed opacity-50'
              : 'cursor-pointer'
          ]"
          @click="!btn.disabled && toggleButton(idx)"
        >
          <v-tooltip
            v-if="btn.tooltip"
            location="top"
            :theme="theme"
            open-delay="400"
          >
            {{ btn.tooltip }}
            <template #activator="{ props: tooltipProps }">
              <p
                v-bind="tooltipProps"
                class="text-xs"
              >
                {{ btn.name }}
              </p>
            </template>
          </v-tooltip>
          <p
            v-else
            class="text-xs"
          >
            {{ btn.name }}
          </p>
        </button>
        <!-- Divider between buttons -->
        <v-divider
          v-if="idx < buttonItems.length - 1"
          vertical
          inset
        />
      </template>

      <!-- Divider before menu, if menu exists -->
      <v-divider
        v-if="buttonsMenu?.length"
        vertical
        inset
      />
      <v-menu
        v-if="buttonsMenu?.length"
        offset-y
        :theme="theme"
        :disabled="disabled"
      >
        <template #activator="{ props: menuProps }">
          <div
            v-bind="menuProps"
            class="px-0 flex items-center justify-center h-full"
            :class="theme === 'dark' ? 'text-white' : 'text-black'"
          >
            <v-icon class="self-center">
              mdi-menu-right
            </v-icon>
          </div>
        </template>
        <v-list class="py-0">
          <v-list-item
            v-for="item in buttonsMenu"
            :key="item.name"
            @click="item.action()"
          >
            <v-list-item-title>{{ item.name }}</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, watch } from 'vue'

/**
 * One button in the group
 */
interface ButtonItem {
  /** Button name */
  name: string
  /** Tooltip to be displayed on mouse hover */
  tooltip?: string
  /** If the group is a switch type, this button should be pre-selected on initialization */
  preSelected?: boolean
  /** Optional color for the button when it is active (default blue #0B5087) */
  activeColor?: string
  /** selectively disable the button */
  disabled?: boolean
}

/**
 * One menu action behind the “more” arrow
 */
interface MenuItem {
  /** Name of the action */
  name: string
  /** Action to be executed when the item is clicked */
  action: () => void
}

const props = defineProps<{
  /** 
   * The array of buttons to be shown. 
   * Object is composed by name, tooltip, preSelected, activeColor, disabled  
   */
  buttonItems: ButtonItem[]
  /** Optional “more” menu actions */
  buttonsMenu?: MenuItem[]
  /** Disable all interaction */
  disabled?: boolean
  /** light or dark theme (d)efault light) */
  theme?: 'light' | 'dark'
  /** Type of button group: 'switch' enforces single select, 'toggle' allows multiple */
  type: 'switch' | 'toggle'
  /** height of the container (default 30px) */
  height?: string
}>()

const emit = defineEmits<{
  (e: 'update:selected', value: boolean[]): void
}>()

const selected = ref<boolean[]>([])

watch(
  () => props.buttonItems,
  items => {
    if (props.type === 'switch') {
      const first = items.findIndex(b => b.preSelected)
      const idx = first >= 0 ? first : 0
      selected.value = items.map((_, i) => i === idx)
    } else {
      selected.value = items.map(b => !!b.preSelected)
    }
    emit('update:selected', selected.value)
  },
  { immediate: true }
)

function toggleButton(idx: number) {
  if (props.disabled) return
  if (props.type === 'switch') {
    if (!selected.value[idx]) {
      selected.value = selected.value.map((_, i) => i === idx)
    }
  } else {
    selected.value[idx] = !selected.value[idx]
  }
  emit('update:selected', selected.value)
}
</script>
