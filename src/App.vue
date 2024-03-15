<script setup>
import { ref, computed } from "vue";
import PocketBase from "pocketbase";

const pb = new PocketBase("https://petriknap.fun");

const events = ref([]);
const loading = ref(true);
const error = ref("");
const categories = ref([]);
const searchQuery = ref("");
const selectedCategory = ref("");

const formatTime = (time) => {
  const date = new Date(time);
  const hours = date.getHours().toString().padStart(2, "0");
  const minutes = date.getMinutes().toString().padStart(2, "0");
  return `${hours}:${minutes}`;
};

const fetchCategories = async () => {
  try {
    categories.value = await pb.collection("categories").getFullList();
  } catch (e) {
    error.value = e.message;
  }
};

const fetchEvents = async () => {
  try {
    events.value = await pb.collection("events").getFullList({
      expand: "location, category",
    });
  } catch (e) {
    error.value = e.message;
  } finally {
    loading.value = false;
  }
};

fetchCategories();
fetchEvents();

const filteredEvents = computed(() => {
  let filtered = events.value;

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    filtered = filtered.filter((event) => {
      return (
        event.name.toLowerCase().includes(query) ||
        event.description.toLowerCase().includes(query)
      );
    });
  }

  if (selectedCategory.value) {
    filtered = filtered.filter((event) => {
      return event.expand.category.name === selectedCategory.value;
    });
  }

  return filtered;
});
</script>

<template>
  <header class="py-4 text-center sticky top-0 bg-[#181825] shadow-lg">
    <h1
      class="text-transparent bg-clip-text bg-gradient-to-r from-[#A6E3A1] to-[#89DCEB] font-black text-4xl mb-4"
    >
      Petrik Nap
    </h1>
    <!-- <hr class="border-[#CBA6F7] mb-2" /> -->
    <div class="flex flex-row justify-center items-center gap-1">
      <input
        type="text"
        class="bg-[#1E1E2E] p-2 rounded-lg mt-2 text-white border-none"
        placeholder="Keresés..."
        v-model="searchQuery"
      />
      <select
        class="bg-[#1E1E2E] p-2 rounded-lg mt-2 text-white border-none"
        v-model="selectedCategory"
      >
        <option value="">-</option>
        <option v-for="category in categories" :key="category.id">
          {{ category.name }}
        </option>
      </select>
    </div>
  </header>
  <main>
    <div v-if="loading" class="flex justify-center items-center">
      <div
        class="inline-block h-8 w-8 animate-spin rounded-full border-4 border-solid border-current border-e-transparent align-[-0.125em] text-surface motion-reduce:animate-[spin_1.5s_linear_infinite] dark:text-white"
        role="status"
      >
        <span
          class="!absolute !-m-px !h-px !w-px !overflow-hidden !whitespace-nowrap !border-0 !p-0 ![clip:rect(0,0,0,0)]"
          >Betöltés...</span
        >
      </div>
    </div>
    <h1 class="text-center text-red-400" v-if="error">Hiba: {{ error }}</h1>
    <div v-else>
      <div
        v-for="event in filteredEvents"
        :key="event.id"
        class="bg-[#1E1E2E] p-4 rounded-lg m-4"
      >
        <h2 class="text-2xl font-bold text-slate-200">
          {{ event.name }} | {{ event.expand.location.name }}
        </h2>
        <p class="text-slate-200 mb-2">{{ event.description }}</p>
        <label for="my_modal_6" class="btn bg-[#CBA6F7] text-black"
          >Több infó</label
        >
        <input type="checkbox" id="my_modal_6" class="modal-toggle" />
        <div class="modal" role="dialog">
          <div class="modal-box bg-[#1E1E2E]">
            <h3 class="font-bold text-lg text-slate-100">
              {{ event.name }} | {{ event.expand.location.name }} |
              {{ formatTime(event.start) }} -
              {{ formatTime(event.end) }}
            </h3>
            <p class="text-slate-100">
              Kategória: {{ event.expand.category.name }}
            </p>
            <p class="pb-2 text-slate-100">
              Szervező(k): {{ event.organizers }}
            </p>
            <p class="text-slate-100">{{ event.description }}</p>
            <div class="modal-action">
              <label for="my_modal_6" class="btn bg-[#CBA6F7] text-black"
                >Bezárás</label
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
