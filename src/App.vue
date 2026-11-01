<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";

// Type of one drama row (same columns as the "dramas" table)
type Drama = {
  id: number;
  title: string;
  channel: string | null;
  episodes: number | null;
  watched: boolean;
  created_at: string;
};

// Base URL of the backend, from the .env file
const baseURL = import.meta.env.VITE_BASE_URL;

// ---------- State ----------
const dramas = ref<Drama[]>([]); // the list shown on the page
const titleText = ref(""); // v-model of the title input
const channelText = ref(""); // v-model of the channel input
const episodesText = ref(""); // v-model of the episodes input
const filter = ref("all"); // "all" | "false" (not watched) | "true" (watched)
const edit = ref(false); // true = the form is in edit mode
const currentDrama = ref<Drama | null>(null); // the drama being edited
const errorMessage = ref(""); // error text shown on the page

// Show a readable message from the backend (400, 404, 500 ...)
function showError(err: unknown) {
  if (axios.isAxiosError(err)) {
    errorMessage.value =
      err.response?.data?.error ?? "Cannot connect to the server";
  } else {
    errorMessage.value = "Something went wrong";
  }
}

// ---------- READ ----------
async function fetchDramas() {
  try {
    let url = `${baseURL}/dramas`;
    if (filter.value !== "all") {
      url = `${baseURL}/dramas?watched=${filter.value}`;
    }
    const res = await axios.get<Drama[]>(url);
    dramas.value = res.data;
  } catch (err) {
    showError(err);
  }
}

function changeFilter(newFilter: string) {
  filter.value = newFilter;
  fetchDramas();
}

// ---------- CREATE + UPDATE (same form) ----------
async function handleSubmitDrama() {
  if (titleText.value.trim() === "") {
    errorMessage.value = "Please type a drama title";
    return;
  }

  // Send null when the optional boxes are empty
  const data = {
    title: titleText.value,
    channel: channelText.value,
    episodes: episodesText.value === "" ? null : Number(episodesText.value),
  };

  try {
    if (!edit.value) {
      await axios.post(`${baseURL}/dramas`, data); // CREATE
    } else if (currentDrama.value) {
      await axios.put(`${baseURL}/dramas/${currentDrama.value.id}`, data); // UPDATE
    }
    resetForm();
    fetchDramas();
  } catch (err) {
    showError(err);
  }
}

function handleEditDrama(drama: Drama) {
  edit.value = true;
  currentDrama.value = drama;
  titleText.value = drama.title;
  channelText.value = drama.channel ?? "";
  episodesText.value = drama.episodes === null ? "" : String(drama.episodes);
  errorMessage.value = "";
}

function resetForm() {
  edit.value = false;
  currentDrama.value = null;
  titleText.value = "";
  channelText.value = "";
  episodesText.value = "";
  errorMessage.value = "";
}

// ---------- UPDATE (watched on/off) ----------
async function toggleWatched(drama: Drama) {
  try {
    await axios.patch(`${baseURL}/dramas/${drama.id}`, {
      watched: !drama.watched,
    });
    fetchDramas();
  } catch (err) {
    showError(err);
  }
}

// ---------- DELETE ----------
async function handleDeleteDrama(id: number) {
  try {
    await axios.delete(`${baseURL}/dramas/${id}`);
    fetchDramas();
  } catch (err) {
    showError(err);
  }
}

// Load the dramas when the page opens
fetchDramas();
</script>

<template>
  <div class="page">
    <div class="card">
      <p class="small-title">DRAMA NOTES</p>
      <h1>My Drama Watchlist</h1>
      <p class="subtitle">What should I watch next?</p>

      <!-- Form: Add / Update -->
      <div class="form">
        <input
          v-model="titleText"
          type="text"
          placeholder="Type a drama title"
          @keyup.enter="handleSubmitDrama"
        />
        <div class="input-row">
          <input v-model="channelText" type="text" placeholder="Channel" />
          <input
            v-model="episodesText"
            type="number"
            min="1"
            placeholder="Episodes"
          />
          <button class="add-btn" @click="handleSubmitDrama">
            <span>{{ edit ? "✓" : "+" }}</span>
            {{ edit ? "Update" : "Add" }}
          </button>
        </div>
        <button v-if="edit" class="cancel-btn" @click="resetForm">
          Cancel editing
        </button>
      </div>

      <div class="typing-box">Currently typing: {{ titleText }}</div>

      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>

      <!-- Header + filter (calls GET /dramas?watched=...) -->
      <div class="watchlist-header">
        <h2>Watchlist ({{ dramas.length }})</h2>
        <div class="filters">
          <button
            class="filter-btn"
            :class="{ active: filter === 'all' }"
            @click="changeFilter('all')"
          >
            All
          </button>
          <button
            class="filter-btn"
            :class="{ active: filter === 'false' }"
            @click="changeFilter('false')"
          >
            To watch
          </button>
          <button
            class="filter-btn"
            :class="{ active: filter === 'true' }"
            @click="changeFilter('true')"
          >
            Watched
          </button>
        </div>
      </div>

      <p v-if="dramas.length === 0" class="empty-message">
        Nothing here yet. Add your first drama!
      </p>

      <!-- List -->
      <div v-for="drama in dramas" :key="drama.id" class="drama-item">
        <div class="drama-info">
          <span class="drama-title" :class="{ done: drama.watched }">
            {{ drama.title }}
          </span>
          <span class="drama-meta">
            {{ drama.channel ?? "-" }} ·
            {{ drama.episodes === null ? "?" : drama.episodes }} episodes
          </span>
        </div>

        <div class="actions">
          <button
            class="watch-btn"
            :class="{ active: drama.watched }"
            @click="toggleWatched(drama)"
          >
            {{ drama.watched ? "Watched ✓" : "Mark watched" }}
          </button>
          <button class="edit-btn" @click="handleEditDrama(drama)">Edit</button>
          <button class="delete-btn" @click="handleDeleteDrama(drama.id)">
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
