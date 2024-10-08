<script setup lang="ts">
import { definePageMeta, navigateTo, useHead } from "#imports"

import { computed, onMounted, ref } from "vue"

import {
  ALL_MEDIA,
  isAdditionalSearchType,
  isSupportedMediaType,
  SearchType,
} from "~/constants/media"
import { useAnalytics } from "~/composables/use-analytics"

import { useMediaStore } from "~/stores/media"
import { useSearchStore } from "~/stores/search"
import { useUiStore } from "~/stores/ui"
import { useFeatureFlagStore } from "~/stores/feature-flag"

import VDarkModeFeatureNotice from "~/components/VFeatureNotice/VDarkModeFeatureNotice.vue"
import VHomeGallery from "~/components/VHomeGallery/VHomeGallery.vue"
import VHomepageContent from "~/components/VHomepageContent.vue"

defineOptions({
  name: "HomePage",
})

definePageMeta({
  layout: "default",
})

const featureFlagStore = useFeatureFlagStore()
const mediaStore = useMediaStore()
const searchStore = useSearchStore()
const uiStore = useUiStore()

const { sendCustomEvent } = useAnalytics()

useHead({
  meta: [{ hid: "theme-color", name: "theme-color", content: "#ffe033" }],
})

/**
 * Reset the search type, search term and filters when the user navigates [back] to the homepage.
 */
onMounted(() => {
  searchStore.$reset()
  mediaStore.$reset()
})

const isXl = computed(() => uiStore.isBreakpoint("xl"))

const isDarkModeSeen = computed(() => uiStore.isDarkModeSeen)

const searchType = ref<SearchType>(ALL_MEDIA)

const setSearchType = (type: SearchType) => {
  if (type === ALL_MEDIA || isSupportedMediaType(type)) {
    searchType.value = type
  } else if (
    featureFlagStore.isOn("additional_search_types") &&
    isAdditionalSearchType(type)
  ) {
    searchType.value = type
  }
}

const showThemeSwitcher = computed(() =>
  featureFlagStore.isOn("dark_mode_ui_toggle")
)

const handleSearch = (searchTerm: string) => {
  sendCustomEvent("SUBMIT_SEARCH", {
    searchType: searchType.value,
    query: searchTerm,
  })

  return navigateTo(
    searchStore.updateSearchPath({
      type: searchType.value,
      searchTerm,
    })
  )
}
</script>

<template>
  <main
    class="index flex w-full flex-shrink-0 flex-grow flex-col justify-center gap-6 px-6 sm:px-0 xl:flex-row xl:items-center xl:gap-0"
  >
    <div
      class="flex flex-grow flex-col items-center justify-center gap-6 xl:h-[33rem] xl:flex-grow-0 xl:items-start"
    >
      <VDarkModeFeatureNotice
        v-if="showThemeSwitcher && !isDarkModeSeen"
        class="sm:mx-14 md:mx-20 lg:mx-26 xl:me-0"
      />

      <VHomepageContent
        class="my-auto sm:px-14 md:px-20 lg:px-26 xl:w-[53.375rem] xl:pe-0"
        :handle-search="handleSearch"
        :search-type="searchType"
        :set-search-type="setSearchType"
      />
    </div>

    <!-- Image carousel -->
    <VHomeGallery v-if="isXl" class="mx-10 me-12 flex h-full flex-grow" />
  </main>
</template>

<style>
@screen lg {
  .homepage-images {
    transform: translateY(-7.143vh);
  }

  .homepage-image:nth-child(even) {
    transform: translateY(50%);
  }
}

.homepage-image {
  transition-delay: var(--transition-index) !important;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: 0.5s;
}
</style>
