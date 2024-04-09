<script>
const GITHUB_ENDPOINT = "https://api.github.com/repos/";
const API_PATH = "/core/commits?per_page=30&sha=main";

export default {
  
  data: () => ({
    repo: "vuejs",
    commits: null,
    error: null,
    loading: false,
  }),
  mounted() {
    this.fetchRepoInfo();
  },
  watch: {
    repo: "fetchRepoInfo",
    
  },
  methods: {
    fetchRepoInfo() {
      this.commits = null;
      this.loading = true;
      this.error = null;
      fetch(GITHUB_ENDPOINT + this.repo + API_PATH)
        .then((res) => {
          if(res.status !== 200) {
              throw ("An error occured!")
          }
        })
        .then((result) => {
          this.commits = result;
          this.loading = false;
        })
        .catch((err) => {
          console.error(err);
          this.error = err;
          this.loading = false;
        });
    },
    computed: {
  console: () => console,
    },
    truncate(v) {
      if (!v) return;
      const newline = v.indexOf("\n");
      return newline > 0 ? v?.slice(0, newline) : v;
    },
    formatDate(v) {
      return v.replace(/T|Z/g, " ");
    },
    updateRepoItem(e) {
      this.repo = e.target.value;
    },
  },
};
</script>

<template>
  <input :value="repo" @input="updateRepoItem" />
  <div class="commits">
    <div class="commits-container">
      <div v-if="loading && !error">Loading...</div>
      <div v-else-if="!error && commits">
        <ul>
          <li
            v-for="{ html_url, sha, author, commit } in commits"
            class="commit-container"
          >
            <a :href="html_url" target="_blank" class="commit">
              {{ sha.slice(0, 7) }}</a
            >
            <br />
            <span class="commit-message">
              {{ truncate(commit.message) }}
            </span>
            <br />
            <span class="author">
              <a :href="author.html_url" target="_blank">
                {{ commit.author.name }}
              </a>
            </span>
            <br />
            <span class="date">
              {{ commit.author.date }}
            </span>
          </li>
        </ul>
      </div>
      <div v-else>{{ error }}</div>
    </div>
  </div>
</template>

<style scoped>
.commits {
  width: 100%;
  background-color: "white";
  .commits-container {
    flex-direction: row;
    flex: auto;
    justify-content: center;
    max-width: 60%;
  }
  .commit-container {
    background-color: #f69a84;
    border-radius: 5px;
    padding: 10px;
    margin-top: 10px;
    margin-bottom: 10px;
  }
}
</style>
