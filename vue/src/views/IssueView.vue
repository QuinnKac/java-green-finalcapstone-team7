<template>
  <div class="issues">
    <h1>Issues</h1>
    <ul class="issue-list">
      <li v-for="issue in issues" :key="issue.id">
        <div class="issue-item">
          <h2 @click="toggleIssue(issue.id)">{{ issue.name }}</h2>
          <div v-if="!issue.minimized">
            <p>{{ issue.description }}</p>
            <template v-if="isAuthenticated">
              <form @submit.prevent="submitVote(issue.id)" class="vote-form">
                <div class="options">
                  <label class="option" v-if="issue.option1">
                    <input type="radio" v-model="selectedOption.issueID" value="1" class="radio-input">
                    <span class="option-text">{{ issue.option1 }}</span>
                  </label>
                  <label class="option" v-if="issue.option2">
                    <input type="radio" v-model="selectedOption.issueID" value="2" class="radio-input">
                    <span class="option-text">{{ issue.option2 }}</span>
                  </label>
                  <label class="option" v-if="issue.option3">
                    <input type="radio" v-model="selectedOption.issueID" value="3" class="radio-input">
                    <span class="option-text">{{ issue.option3 }}</span>
                  </label>
                  <label class="option" v-if="issue.option4">
                    <input type="radio" v-model="selectedOption.issueID" value="4" class="radio-input">
                    <span class="option-text">{{ issue.option4 }}</span>
                  </label>
                </div>
                <div class="timelimit"><strong>VOTING ENDS {{ formatEndTime(issue.endTime) }}</strong></div>
                <button type="submit" class="submit-button">Submit Vote</button>
                <button type="button" class="view-results-button" @click="showResults(issue.id)">View Results</button>
                <div class='issue-number'>ID#: {{ issue.id }}</div>
                <div class='issue-cat'>Category: {{ issue.groupId }}</div>
              </form>
            </template>
            <template v-if="!isAuthenticated">
              <p class='sign-up'>
                <router-link v-bind:to="{ name: 'register' }">Sign up to vote now!</router-link>
              </p>
            </template>
            <div v-if="getResult(issue.id)">
            <h4>Results:</h4>
            <div v-for="(votes, option) in getResult(issue.id)" :key="option" class="bar-graph">
            <div class="bar" :style="{ width: (votes * 10) + 'px', backgroundColor: getBarColor(issue, option) }"></div>
            <span class="option-text">{{ formatOption(issue, option) }}: {{ votes }} votes</span>
              </div>
            </div>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>




<script>
import IssueService from '../services/IssueService.js';

export default {
  data() {
    return {
      issues: [],
      selectedOption: { issueID: 0 },
      results: [],
      colors: ['#FF5733', '#33FF57', '#5733FF', '#33FFFF', '#FF33FF', '#FFFF33', '#3399FF', '#FF3333', '#33FF99', '#FF9933'],
    };
  },
  created() {
    this.loadIssues();
  },
  computed: {
    isAuthenticated() {
      return this.$store.getters.isAuthenticated;
    }
  },
  methods: {
    loadIssues() {
      IssueService.getIssues()
        .then(response => {
          this.issues = response.data.map(issue => ({
            ...issue,
            minimized: true
          }));
        })
        .catch(error => {
          console.error('Error loading issues:', error);
        });
    },
    getBarColor(issue, option) {
      const index = Object.keys(this.getResult(issue.id)).indexOf(option);
      return this.colors[index % this.colors.length];
    },
    submitVote(issueId) {
      const selectedOption = this.selectedOption.issueID;
      const userId = this.$store.state.user.id;
      IssueService.submitVote(issueId, selectedOption, userId)
        .then(() => {
          console.log('Vote submitted successfully');
          this.showResults(issueId);
        })
        .catch(error => {
          console.error('Error submitting vote:', error);
          alert('Failed to submit vote. Please try again later.');
        });
    },
    toggleIssue(issueId) {
      const issue = this.issues.find(issue => issue.id === issueId);
      if (issue) {
        issue.minimized = !issue.minimized;
      }
    },
    showResults(issueId) {
      IssueService.getResults(issueId)
        .then(response => {
          const resultIndex = this.results.findIndex(result => result.issueId === issueId);
          const newResult = { issueId, data: response.data };
          if (resultIndex !== -1) {
            this.$set(this.results, resultIndex, newResult);
          } else {
            this.results.push(newResult);
          }
        })
        .catch(error => {
          console.error('Error loading results:', error);
        });
    },
    getResult(issueId) {
      const result = this.results.find(result => result.issueId === issueId);
      return result ? result.data : null;
    },
    formatOption(issue, option) {
      switch (option) {
        case 'option1':
          return issue.option1;
        case 'option2':
          return issue.option2;
        case 'option3':
          return issue.option3;
        case 'option4':
          return issue.option4;
        default:
          return 'Unknown Option';
      }
    },
    formatEndTime(endTime) {
      const date = new Date(Date.parse(endTime));
      return date.toLocaleString();
    }
  }
};
</script>



<style scoped>

.issues {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh; 
  background-color: #E4E4E4;
}
.issue-view {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: Arial, Helvetica, sans-serif;
}

.issue-list {
  display: flex;
  flex-direction: column;
  align-items: center;
  list-style: none;
  padding: 0;
  width: 100%;
  font-family: Arial, Helvetica, sans-serif;
}

.issue-item {
  background-color: #7ba7bf;
  height: auto;
  width: 400px;
  padding: 40px;
  margin-top: 30px;
  margin-bottom: 30px;
  border-radius: 50px;
  text-align: left;
  box-shadow: 0 5px 25px #63747A;
  position: relative;
}

h2 {
  color: black;
  font-family: Arial, Helvetica, sans-serif;
}

h2:hover{
  color: black;
  text-shadow: 0 0 15px rgba(61, 84, 211, 0.712), 0 0 15px lightblue;
}

.vote-form {
  display: flex;
  flex-direction: column;
  align-items: left;
}

.options {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.option {
  margin-bottom: 10px;
}

.radio-input {
  margin-right: 5px;
}

.submit-button {
  background-color: #FFDB15;
  color: #020301;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin-top: 20px;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.submit-button:hover {
  background-color: #ffa115 ;
}

.view-results-button {
  background-color: #00BFFF;
  color: white;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin-top: 10px;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.view-results-button:hover {
  background-color: #007ACC;
}

.option-text {
  font-size: 16px;
}

.issue-item h3 {
  cursor: pointer;
}

.issue-number {
  position: absolute;
  top: 15px;
  right: 25px;
  padding: 18px;
  border-radius: 5px;
}

.issue-cat {
  position: absolute;
  top: 15px;
  left: 25px;
  padding: 18px;
  border-radius: 5px;
}

.bar-graph {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.bar {
  height: 20px;
  background-color: #0b53a0; /* Adjust color as needed */
  margin-right: 5px;
}

.option-text {
  margin-left: 10px;
  font-size: 14px;
}

h1 {
  font-family: Arial, Helvetica, sans-serif;
}

.timelimit {
  display: flex;
  justify-content: center;
  align-items: center;
  padding-top: 20px;
}
</style>

