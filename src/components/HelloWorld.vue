<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue';
import './HelloWorld.css';

// Define the types for user objects
interface User {
  id: string;
  name: string;
  email: string;
  location: string;
  gender: string;
  picture: string;
}

const username = ref('');
const searchedUsersList = ref<User[]>([]);
const selectedUser = ref<User | null>(null);
const loadingMore = ref(false);
const resultsPerPage = 25;
const filterText = ref('');
const filterGender = ref('');
let currentPage = 1;

// Save the current page state to localStorage
function savePageState() {
  localStorage.setItem('pageState', JSON.stringify({
    searchedUsersList: searchedUsersList.value,
    selectedUser: selectedUser.value,
    filterText: filterText.value,
    filterGender: filterGender.value,
  }));
}

// Load the saved page state from localStorage
function loadPageState() {
  const savedState = localStorage.getItem('pageState');
  if (savedState) {
    const { searchedUsersList: savedSearchedUsersList, selectedUser: savedSelectedUser, filterText: savedFilterText, filterGender: savedFilterGender } = JSON.parse(savedState);
    searchedUsersList.value = savedSearchedUsersList;
    selectedUser.value = savedSelectedUser;
    filterText.value = savedFilterText;
    filterGender.value = savedFilterGender;
  }
}

// Watch for changes in the page state and save to localStorage
onMounted(() => {
  loadPageState();
});

watch([searchedUsersList, selectedUser, filterText, filterGender], () => {
  savePageState();
});

async function fetchUsers(results: number, page: number) {
  const response = await fetch(`https://randomuser.me/api/?results=${results}&nat=us&seed=${username.value}&page=${page}`);
  const data = await response.json();
  return data.results.map((result: any) => ({
    id: result.login.uuid,
    name: `${result.name.first} ${result.name.last}`,
    email: result.email,
    phone: result.phone,
    location: `${result.location.city}, ${result.location.state}, ${result.location.country}`,
    gender: result.gender,
    picture: result.picture.thumbnail,
  }));
}

async function searchUser() {
  currentPage = 1;
  searchedUsersList.value = [];
  await fetchUsers(resultsPerPage, currentPage);
}

async function loadMoreResults() {
  loadingMore.value = true;
  currentPage++;
  const newResults = await fetchUsers(resultsPerPage, currentPage);
  searchedUsersList.value = searchedUsersList.value.concat(newResults);
  loadingMore.value = false;
}

onMounted(async () => {
  searchedUsersList.value = await fetchUsers(resultsPerPage, currentPage);
});

const filteredUsers = computed(() => {
  let filtered = searchedUsersList.value;

  if (filterText.value) {
    const searchTerm = filterText.value.toLowerCase();
    filtered = filtered.filter(
      (user) =>
        user.name.toLowerCase().includes(searchTerm) ||
        user.email.toLowerCase().includes(searchTerm)
    );
  }

  if (filterGender.value) {
    filtered = filtered.filter((user) => user.gender === filterGender.value);
  }

  return filtered;
});

function formatName(name: string): string {
  const [firstName, lastName] = name.split(" ");
  return `${capitalize(firstName)} ${capitalize(lastName)}`;
}

function capitalize(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

function selectUser(user: User) {
  selectedUser.value = user;
}

function closeUserDetails() {
  selectedUser.value = null; // Clear the selectedUser to hide the user-details-card
}
</script>

<template>
  <div class="wrapper">
    <div class="content-wrapper">
      <div class="container">
        <h1>Search for a User</h1>
        <div class="inputs">
          <input type="text" v-model="filterText" placeholder="Filter by name" />
          <select v-model="filterGender">
            <option value="">All</option>
            <option value="male">Male</option>
            <option value="female">Female</option>
          </select>
          <button @click="searchUser" class="search">Search</button>
        </div>

        <div class="user-container">
          <ul>
            <li v-for="(user, index) in filteredUsers" :key="user.id" @click="selectUser(user)">
              <div class="user-info">
                <div class="user-thumbnail">
                  <img :src="user.picture" :alt="user.name" />
                </div>
                <div class="user-details">
                  <div class="user-name">{{ formatName(user.name) }}</div>
                  <div class="user-email">{{ user.email }}</div>
                </div>
              </div>
              <template v-if="index === filteredUsers.length - 1 && !loadingMore">
                <div class="more-results-button">
                  <button @click="loadMoreResults">More results...</button>
                </div>
              </template>
            </li>
          </ul>
          <div v-if="loadingMore">Loading more results...</div>
        </div>
      </div>

      <div class="container-card" v-if="selectedUser">
        <div class="user-details-card fixed">
          <div class="user-card">
            <div class="user-thumbnail">
              <img :src="selectedUser.picture" :alt="selectedUser.name" />
            </div>
            <div class="user-card-details">
              <div class="user-id">
                <span class="label">ID:</span>
                <p>{{ selectedUser.id }}</p>
              </div>
              <div class="user-location">
                <span class="label">Location:</span>
                <p>{{ selectedUser.location }}</p>
              </div>
              <div class="user-contact">
                <span class="label">Contact: </span>
                <p>{{ selectedUser.email }}</p>
              </div>
            </div>
          </div>
          <button class="close-button search" @click="closeUserDetails">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Add your CSS styles here */
</style>
