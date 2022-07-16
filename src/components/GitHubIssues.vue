<template>
  <div class="container">
    <h1 class="title">VueJS | GitHub</h1>
    <p class="lead">
      Aplicação que lista issues de um repositório do GitHub, usando VueJS.
    </p>

    <div v-if="response.status === 'error'" class="alert alert-danger">
      {{ response.message }}
    </div>

    <div class="inputs">
      <div class="col">
        <div class="form-group">
          <input
            v-model="username"
            type="text"
            class="form-control"
            placeholder="Usuário GitHub"
          />
        </div>
      </div>

      <div class="col">
        <div class="form-group">
          <input
            v-model="repository"
            type="text"
            class="form-control"
            placeholder="Repositório GitHub"
          />
        </div>
      </div>

      <div class="col-3">
        <div class="form-group">
          <button @click.prevent.stop="getIssues()" class="btn btn-success">
            ->
          </button>
          <button @click.prevent.stop="reset()" class="btn btn-danger">
            X
          </button>
        </div>
      </div>
    </div>

    <br />
    <hr />
    <br />

    <table class="table table-sm table-bordered">
      <thead>
        <tr>
          <th>Número</th>
          <th>Título</th>
        </tr>
      </thead>

      <tbody>
        <tr v-if="loader.getIssues">
          <td class="text-center" colspan="2">
            <img src="/static/loading.svg" alt="" />
          </td>
        </tr>

        <tr v-for="issue in issues" :key="issue.number">
          <td>
            <router-link
              :to="{
                name: 'GitHubIssue',
                params: {
                  name: username,
                  repo: repository,
                  issue: issue.number,
                },
              }"
            >
              {{ issue.number }}
            </router-link>
          </td>

          <td>{{ issue.title }}</td>
        </tr>

        <tr v-if="noIssues">
          <td class="text-center" colspan="2">Nenhuma issue encontrada!</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
/* eslint-disable */
import axios from "axios";

export default {
  name: "GitHubIssues",

  created() {
    this.getLocalData();
  },

  data() {
    return {
      username: "",
      repository: "",
      issues: [],
      response: {
        status: "",
        message: "",
      },
      loader: {
        getIssues: false,
      },
    };
  },

  computed: {
    showIssues() {
      return !!this.issues.length && !this.loader.getIssues;
    },

    noIssues() {
      return !this.issues.length && !this.loader.getIssues;
    },
  },

  methods: {
    reset() {
      this.username = "";
      this.repository = "";
      localStorage.removeItem("gitHubIssues");
    },

    resetResponse() {
      this.response.status = "";
      this.response.message = "";
    },

    getIssues() {
      this.resetResponse();
      this.issues = [];

      if (this.username && this.repository) {
        localStorage.setItem(
          "gitHubIssues",
          JSON.stringify({
            username: this.username,
            repository: this.repository,
          })
        );
        this.loader.getIssues = true;
        const url = `https://api.github.com/repos/${this.username}/${this.repository}/issues`;
        axios
          .get(url)
          .then((response) => {
            this.issues = response.data;
          })
          .catch((error) => {
            // eslint-disable-next-line
            console.log(error.response.data);
            this.response.status = "error";
            this.response.message = "Repositório não existe!";
          })
          .finally(() => {
            this.loader.getIssues = false;
          });
      }
    },

    getLocalData() {
      const localData = JSON.parse(localStorage.getItem("gitHubIssues"));
      if (localData && localData.username && localData.repository) {
        this.username = localData.username;
        this.repository = localData.repository;
        this.getIssues();
      }
    },
  },
};
</script>

<style>
body {
  background-image: linear-gradient(to right, grey, silver);
}

.title {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 5rem;
  margin-top: 2rem;
}

.lead {
  margin-bottom: 2rem;
  margin-top: 8rem;
  text-align: center;
}

.form-group {
  display: flex;
  align-content: center;
  justify-content: center;
  margin-bottom: 1rem;
  text-align: center;
}

.inputs {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 2rem;
}

button {
  margin-left: 0.5rem;
  margin-right: 0.5rem;
}

th {
  text-align: center;
}
</style>
