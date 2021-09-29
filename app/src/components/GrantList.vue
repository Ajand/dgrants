<template>
  <!-- General filters -->
  <BaseFilterNav :items="grantRegistryListNav" :button="button" title="grants:" />
  <ul class="base-grid">
    <li v-for="grant in sortedGrants" :key="grant.id.toString()">
      <GrantCard
        :id="grant.id"
        :name="(grantMetadataResolved && grantMetadataResolved[grant.metaPtr]?.name) || ''"
        :ownerAddress="grant.owner"
        :imgurl="(grantMetadataResolved && grantMetadataResolved[grant.metaPtr]?.logoURI) || '/placeholder_grant.svg'"
      />
    </li>
  </ul>
</template>

<script lang="ts">
import { computed, defineComponent, onUnmounted, PropType, ref, watch } from 'vue';
// --- App Imports ---
import BaseFilterNav from 'src/components/BaseFilterNav.vue';
import GrantCard from 'src/components/GrantCard.vue';
// --- Store ---
import useCartStore from 'src/store/cart';
// --- Types ---
import { FilterNavButton, FilterNavItem, Grant, GrantMetadataResolution } from '@dgrants/types';

type SortingMode = 'newest' | 'oldest' | 'shuffle';

const defaultSortingMode = 'newest';
const grantsSortingMode = ref<SortingMode>(defaultSortingMode);

function useSortedGrants(grants: Grant[]) {
  const sortedGrants = ref(grants);

  function sortGrants(order: SortingMode) {
    grantsSortingMode.value = order;
    if (order === 'newest') sortedGrants.value = sortedGrants.value.sort((a, b) => (a.id < b.id ? 1 : -1));
    else if (order === 'oldest') sortedGrants.value = sortedGrants.value.sort((a, b) => (a.id > b.id ? 1 : -1));
    else sortedGrants.value = sortedGrants.value.sort(() => (Math.random() < 0.5 ? 1 : -1));
  }

  sortGrants(grantsSortingMode.value);

  const grantRegistryListNav = computed(
    () =>
      <FilterNavItem[]>[
        {
          label: 'Sort',
          tag: grantsSortingMode.value,
          menu: [
            {
              label: 'newest',
              action: () => {
                sortGrants('newest');
              },
            },
            {
              label: 'oldest',
              action: () => {
                sortGrants('oldest');
              },
            },
            {
              label: 'shuffle',
              action: () => {
                sortGrants('shuffle');
              },
            },
          ],
        },
      ]
  );

  return {
    sortedGrants,
    grantRegistryListNav,
  };
}

export default defineComponent({
  name: 'GrantList',
  components: { BaseFilterNav, GrantCard },
  props: {
    // --- Required props ---
    grants: { type: Array as PropType<Grant[]>, required: true },
    grantMetadata: { type: Object as PropType<Record<string, GrantMetadataResolution>>, required: true },
    button: { type: Object as PropType<FilterNavButton>, required: false },
  },
  setup(props) {
    const { addToCart, isInCart, removeFromCart } = useCartStore();

    const sortedGrants = ref<Grant[]>([]);
    const grantRegistryListNav = ref<FilterNavItem[]>([]);
    const grantMetadataResolved = ref<Record<string, GrantMetadataResolution>>({});
    // watch for changes on the grants and call useSortedGrants again
    watch(
      () => [props.grants, props.grantMetadata],
      () => {
        const { sortedGrants: _sortedGrants, grantRegistryListNav: _grantRegistryListNav } = useSortedGrants([
          ...props.grants,
        ]);
        sortedGrants.value = _sortedGrants.value;
        grantRegistryListNav.value = _grantRegistryListNav.value;
        grantMetadataResolved.value = props.grantMetadata;
      },
      { immediate: true }
    );

    onUnmounted(() => {
      grantsSortingMode.value = defaultSortingMode;
    });

    return {
      sortedGrants,
      grantRegistryListNav,
      grantMetadataResolved,
      isInCart,
      addToCart,
      removeFromCart,
    };
  },
});
</script>
