<script>
const GITHUB_ENDPOINT = "https://api.github.com/repos/";
const API_PATH = "/core/commits?per_page=22&sha=main";
import { debounce } from "../utils/helpers";
import TopContributor from "./TopContributor.vue";

export default {
  components: {
    TopContributor,
  },
  data: () => ({
    repo: "vuejs",
    visibleCommits: null,
    commits: null,
    error: null,
    loading: false,
    contributionMap: new Map(),
    currentPage: 0,
    pageSize:  10,
  }),
  computed: {
    isNextDisabled(){
      return this.currentPage === (Math.ceil(this.commits.length / this.pageSize)) - 1 ;
      
    },
    isPreviousDisabled(){
      return this.currentPage === 0;
    },
    currentPageInfo(){
     
      const isLastPage = this.isNextDisabled;
      console
      if (isLastPage ){
        return `Showing ${ (((this.currentPage) * this.pageSize )) + (this.commits.length % this.pageSize) } out of ${this.commits.length}`
      }
      return `Showing ${((this.currentPage+1) * this.pageSize)} out of ${this.commits.length}`
    }
  },

  mounted() {
    this.updateRepoItem = debounce((e) => {
      this.repo = e.target.value;
    }, 600);
    this.fetchRepoInfo();
  },

  watch: {
    repo: "fetchRepoInfo",

  },
  methods: {
    increasePage() {
      const nextPageStartIndex = (this.currentPage + 1) * this.pageSize;
      if (nextPageStartIndex < this.commits.length) {
        this.inNextBlocked = false
        this.currentPage++;
        this.visibleCommits =  this.commits.slice(
          nextPageStartIndex,
          nextPageStartIndex + this.pageSize,
        );
      } 
    },
    decreasePage() {
      const prevPageStartIndex = Math.max(
        0,
        (this.currentPage - 1) * this.pageSize,
      );
      if (prevPageStartIndex !== this.currentPage * this.pageSize) {
        this.currentPage--;
        this.visibleCommits =  this.commits.slice(
          prevPageStartIndex,
          prevPageStartIndex + this.pageSize,
        );
      }
    },
    fetchRepoInfo() {
      this.commits = null;
      this.loading = true;
      this.error = null;
      fetch(GITHUB_ENDPOINT + this.repo + API_PATH)
        .then((res) => {
          if (res.status !== 200) {
            throw "An error occured!";
          }
          return res.json();
        })
        .then((result) => {
          this.commits = result;
          this.visibleCommits = result.slice(0,this.pageSize)
          result.map((x) => {
            if (!this.contributionMap.get(x.author.login)) {
              this.contributionMap.set(x.author.login, 1);
            } else {
              this.contributionMap.set(
                x.author.login,
                this.contributionMap.get(x.author.login) + 1,
              );
            }
          });
          this.loading = false;
        })
        .catch((err) => {
          console.error(err);
          this.error = err;
          this.loading = false;
        });
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
  }
  
};
</script>

<template>
  <div class="centered">
    <p class="title">Github Repo Searcher</p>
    <input :value="repo" @input="updateRepoItem" />
  </div>
  <div class="commits">
    <div class="commits-container">
      <div v-if="loading && !error">Loading...</div>
      <div v-else-if="!error && visibleCommits">
        <TopContributor :contributors="contributionMap" />
        <ul>
          <li
            v-for="{ html_url, sha, author, commit } in visibleCommits"
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
        <div class="pagination-container">
          <div>{{ currentPageInfo }}</div>
    <button v-if="!isPreviousDisabled" @click="decreasePage" class="pagination-button-previous">Previous</button>
    <button v-if="!isNextDisabled" @click="increasePage" class="pagination-button-next">Next</button>
  </div>
      </div>
      <div class="error" v-else>{{ error }}</div>
    </div>
  </div>
  
</template>

<style scoped>
.commits {
  width: 100%;
  flex-wrap: nowrap;
  justify-content: center;
  display: flex;
  background-color: "white";
  .commits-container {
    max-width: 60%;
  }

  .commit-container {
    background-color: #f69a84;
    border-radius: 5px;
    padding: 10px;
    margin-top: 10px;
    margin-bottom: 10px;
  }
  .error {
    color: red;
    font: bold;
  }
}
.title {
  font-size: 2.5rem;
}

.centered {
  display: flex;
  flex-direction: column;
  align-items: center;
  row-gap: 15px;
}

.pagination-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  column-gap: 15px;
  justify-content: center;
}
.pagination-button-next {
  appearance: none;
  border: 1px solid rgba(27, 31, 35, .15);
  border-radius: 6px;
  box-shadow: rgba(27, 31, 35, .1) 0 1px 0;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  font-family: -apple-system,system-ui,"Segoe UI",Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji";
  font-size: 14px;
  font-weight: 600;
  line-height: 20px;
  padding: 6px 16px;
  position: relative;
  text-align: center;
  text-decoration: none;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  background-color: #2ea44f;
  white-space: nowrap;
}
.pagination-button-previous {
  appearance: none;
  border: 1px solid rgba(27, 31, 35, .15);
  border-radius: 6px;
  box-shadow: rgba(27, 31, 35, .1) 0 1px 0;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  font-family: -apple-system,system-ui,"Segoe UI",Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji";
  font-size: 14px;
  font-weight: 600;
  line-height: 20px;
  padding: 6px 16px;
  position: relative;
  text-align: center;
  text-decoration: none;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  background-color: #ff004f;
  white-space: nowrap;
}



</style>
