<template>
  <v-autocomplete
    v-model="selectedId"
    :items="options"
    :loading="loading"
    label="Brand"
    :search-input.sync="search"
    item-text="name"
    item-value="brand_id"
    clearable
    hide-no-data
    @update:search-input="onSearch"
    @change="onSelect"
    @blur="onBlur"
  >
    <template #item="{ item }">
      <div class="d-flex align-center">
        <span class="mr-2">{{ item.name }}</span>
      </div>
    </template>
  </v-autocomplete>
</template>

<script setup>
import { ref, watch } from "vue";
import { autoCompleteBrands } from "../api/productService.js";

const props = defineProps({
  brandId: [Number, String, null],
  brandName: [String, null],
});
const emit = defineEmits(["update:brandId", "update:brandName"]);

const search = ref("");
const options = ref([]);
const loading = ref(false);
const selectedId = ref(props.brandId ?? null);

watch(() => props.brandId, v => selectedId.value = v);

watch(selectedId, v => {
  // if user chose an option, set brandName to matched option's name
  emit("update:brandId", v);
  if (v == null) {
    // cleared selection
    emit("update:brandName", null);
  } else {
    const found = options.value.find(o => o.brand_id === v);
    if (found) emit("update:brandName", found.name);
  }
});

let timer = null;
async function onSearch(q) {
  clearTimeout(timer);
  if (!q || q.length < 1) {
    options.value = [];
    // expose typed name so backend can create brand if user leaves
    emit("update:brandName", q || null);
    return;
  }
  timer = setTimeout(async () => {
    loading.value = true;
    try {
      const res = await autoCompleteBrands(q, 8);
      options.value = res || [];
    } catch (err) {
      options.value = [];
    } finally {
      loading.value = false;
    }
  }, 250);
}

function onSelect(value) {
  // value is brand_id or null
  // watch(selectedId) will set brandName when matched; if unmatched we leave brandName as-is
}

function onBlur() {
  // if no selectedId but search has value -> set brandName so backend can create
  if (!selectedId.value && search.value && search.value.trim()) {
    emit("update:brandName", search.value.trim());
  }
}
</script>
