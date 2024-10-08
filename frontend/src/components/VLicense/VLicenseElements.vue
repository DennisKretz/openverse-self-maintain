<script setup lang="ts">
import { useI18n } from "#imports"

import { computed } from "vue"

import type { License } from "~/constants/license"
import { useDarkMode } from "~/composables/use-dark-mode"
import { useUiStore } from "~/stores/ui"
import { camelCase } from "~/utils/case"
import { getElements } from "~/utils/license"

import VIcon from "~/components/VIcon/VIcon.vue"

const props = withDefaults(
  defineProps<{
    /**
     * the slug of the license
     * @values ALL_LICENSES
     */
    license: License
    /**
     * the size of the icons and text
     */
    size?: "big" | "small"
  }>(),
  {
    size: "big",
  }
)

const { effectiveColorMode } = useDarkMode()

const i18n = useI18n({ useScope: "global" })
const elementNames = computed(() =>
  getElements(props.license).filter((icon) => icon !== "cc")
)

const isSmall = computed(() => props.size === "small")
const uiStore = useUiStore()
const isMobile = computed(() => !uiStore.isDesktopLayout)

const getLicenseDescription = (element: string) => {
  return i18n.t(`browsePage.licenseDescription.${camelCase(element)}`)
}
</script>

<template>
  <ul class="flex flex-col gap-y-2 md:gap-y-4">
    <li
      v-for="element in elementNames"
      :key="element"
      class="flex items-center gap-x-3 text-sm md:text-base"
    >
      <VIcon
        view-box="0 0 30 30"
        :size="isSmall || isMobile ? 5 : 6"
        :name="`licenses/${element}-${effectiveColorMode}`"
      />
      <span v-if="elementNames.length > 1" class="sr-only">{{
        element.toUpperCase()
      }}</span>
      <p class="label-regular" :class="{ 'md:description-regular': !isSmall }">
        {{ getLicenseDescription(element) }}
      </p>
    </li>
  </ul>
</template>
