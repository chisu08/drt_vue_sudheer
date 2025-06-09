<template>
  <div class="space-y-4">
    <!-- Search -->
    <div class="flex space-x-2">
      <input
        v-model="searchTerm"
        @keyup.enter="applySearch"
        placeholder="Name or NORAD Cat ID"
        class="border rounded px-2 py-1 flex-1"
      />
      <button @click="applySearch" class="bg-blue-500 text-white px-4 rounded">
        Search
      </button>
    </div>

    <!-- Filters -->
    <div class="flex space-x-4 items-center">
      <div>
        <label class="font-semibold">Object Types</label>
        <select multiple v-model="selectedObjectTypes" class="border rounded px-2 py-1">
          <option v-for="type in objectTypeOptions" :key="type" :value="type">
            {{ type }}
          </option>
        </select>
      </div>
      <div>
        <label class="font-semibold">Orbit Codes</label>
        <select multiple v-model="selectedOrbitCodes" class="border rounded px-2 py-1">
          <option v-for="code in orbitCodeOptions" :key="code" :value="code">
            {{ code }}
          </option>
        </select>
      </div>
      <button @click="applyFilters" class="bg-green-500 text-white px-4 rounded">
        Apply Filters
      </button>
    </div>

    <!-- Counts -->
    <div>
      <span>Total: {{ counts.total }}</span>
      <span
        v-for="(val, key) in counts"
        v-if="key !== 'total'"
        class="ml-4"
        :key="key"
      >
        {{ key }}: {{ val }}
      </span>
    </div>

    <!-- Virtualized Table -->
    <div class="h-[600px] border">
      <RecycleScroller
        :items="data"
        :item-size="50"
        key-field="noradCatId"
      >
        <template #default="{ item }">
          <div class="grid grid-cols-6 gap-2 px-2 py-1 border-b">
            <div>{{ item.name }}</div>
            <div>{{ item.noradCatId }}</div>
            <div>{{ item.orbitCode }}</div>
            <div>{{ item.objectType }}</div>
            <div>{{ item.countryCode }}</div>
            <div>{{ item.launchDate }}</div>
          </div>
        </template>
      </RecycleScroller>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';
import { RecycleScroller } from 'vue-virtual-scroller';
import type { Satellite } from '../types';

const data = ref<Satellite[]>([]);
const counts = reactive<{ [key: string]: number }>({ total: 0 });
const searchTerm = ref('');
const selectedObjectTypes = ref<string[]>([]);
const selectedOrbitCodes = ref<string[]>([]);
const objectTypeOptions = ['ROCKET BODY', 'DEBRIS', 'UNKNOWN', 'PAYLOAD'];
const orbitCodeOptions = [
  "LEO","LEO1","LEO2","LEO3","LEO4","MEO","GEO","HEO","IGO","EGO","NSO","GTO","GHO","HAO","MGO","LMO","UFO","ESO","UNKNOWN"
];

const fetchData = async () => {
  try {
    const params: any = {
      attributes: 'noradCatId,intlDes,name,launchDate,decayDate,objectType,launchSiteCode,countryCode,orbitCode'
    };
    if (selectedObjectTypes.value.length)
      params.objectTypes = selectedObjectTypes.value.join(',');
    if (selectedOrbitCodes.value.length)
      params.orbitCodes = selectedOrbitCodes.value.join(',');
    if (searchTerm.value) params.search = searchTerm.value;

    const res = await axios.get('https://backend.digantara.dev/v1/satellites', { params });
    data.value = res.data.data;
    Object.assign(counts, res.data.counts);
  } catch (e) {
    console.error('Fetch error:', e);
  }
};

const applySearch = () => fetchData();
const applyFilters = () => fetchData();

onMounted(fetchData);
<\/script>

<style>
@import 'vue-virtual-scroller/dist/vue-virtual-scroller.css';
</style>
