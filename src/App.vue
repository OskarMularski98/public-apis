<template>
  <div class="container-fluid d-flex justify-content-center">
    <div class="row col-8">
      <input
        v-model="search"
        placeholder="Search"
        type="text"
        class="form-control"
      />
      <label for="itemsPerPage">Items per Page </label>
      <select
        name="itemsPerPage"
        @click="changeItemsPerPage"
        v-model="itemsPerPage"
      >
        <option selected :value="10">10</option>
        <option :value="25">25</option>
        <option :value="50">50</option>
      </select>
      <table class="table table-bordered table-striped">
        <thead>
          <tr>
            <th>API</th>
            <th>Auth</th>
            <th>Category</th>
            <th>Cors</th>
            <th>Description</th>
            <th>HTTPS</th>
            <th>Link</th>
          </tr>
        </thead>
        <tbody v-if="!loading">
          <tr v-for="(api, i) in filterApis" :key="i">
            <td>{{ api.API }}</td>
            <td>{{ api.Auth }}</td>
            <td>{{ api.Category }}</td>
            <td>{{ api.Cors }}</td>
            <td>{{ api.Description }}</td>
            <td>{{ api.HTTPS }}</td>
            <td>
              <a :href="api.Link" target="_blank">{{ api.Link }}</a>
            </td>
          </tr>
        </tbody>
        <div v-else>Loading...</div>
      </table>
      <nav>
        <ul class="pagination justify-content-center">
          <li class="page-item" :class="[currentPage <= 1 ? 'disabled' : '']">
            <a class="page-link" href="#" @click="changePage(currentPage - 1)"
              >Previous</a
            >
          </li>
          <!-- <li
              class="page-item"
              v-for="(page, i) in pageNumbers"
              :key="i"
              :class="[currentPage === i + 1 ? 'active' : '']"
            >
              <a class="page-link" href="#" @click="changePage(i + 1)">{{
                i + 1
              }}</a>
            </li> -->
          <!-- <li
          v-for="(page, i) in length"
          :key="i"
          class="page-item"
          :class="[currentPage === i + 1 ? 'active' : '']"
          @click="changePage(i + 1)"
        >
          <a href="#" class="page-link">{{ i + 1 }}</a>
        </li> -->
          <li class="page-item" v-if="currentPage > 3">
            <a
              href="#"
              class="page-link"
              @click="changePage(1)"
              :class="[currentPage === 1 ? 'active' : '']"
              >1</a
            >
          </li>
          <li class="page-item" v-if="currentPage > 4">
            <a class="page-link">...</a>
          </li>
          <li
            class="page-item"
            v-if="currentPage > 2"
            @click="changePage(currentPage - 2)"
          >
            <a class="page-link" href="#"> {{ currentPage - 2 }} </a>
          </li>
          <li
            class="page-item"
            v-if="currentPage > 1"
            @click="changePage(currentPage - 1)"
          >
            <a class="page-link" href="#"> {{ currentPage - 1 }} </a>
          </li>
          <li class="page-item active">
            <a class="page-link" href="#"> {{ currentPage }} </a>
          </li>
          <li
            v-if="currentPage <= pageNumbers.length - 1"
            class="page-item"
            @click="changePage(currentPage + 1)"
          >
            <a class="page-link" href="#"> {{ currentPage + 1 }} </a>
          </li>
          <li
            v-if="currentPage <= pageNumbers.length - 2"
            class="page-item"
            @click="changePage(currentPage + 2)"
          >
            <a class="page-link" href="#"> {{ currentPage + 2 }} </a>
          </li>
          <li class="page-item" v-if="currentPage <= pageNumbers.length - 4">
            <a class="page-link">...</a>
          </li>
          <li class="page-item" v-if="currentPage <= pageNumbers.length - 3">
            <a
              href="#"
              class="page-link"
              @click="changePage(pageNumbers.length)"
              :class="[currentPage === pageNumbers.length ? 'active' : '']"
              >{{ pageNumbers.length }}</a
            >
          </li>
          <li
            class="page-item"
            :class="[currentPage >= pageNumbers.length ? 'disabled' : '']"
          >
            <a class="page-link" href="#" @click="changePage(currentPage + 1)"
              >Next</a
            >
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      loading: false,
      search: "",
      apis: [],
      pageNumbers: [],
      currentPage: 1,
      itemsPerPage: 10,
      currentApis: [],
      indexOfLastApi: null,
      indexOfFirstApi: null,
      length: 10,
      total: null,
    };
  },
  async created() {
    this.loading = true;
    await this.loadApis();
    this.loading = false;
  },
  methods: {
    async loadApis() {
      try {
        const response = await axios.get("https://api.publicapis.org/entries");
        this.total = response.data.count;
        this.apis = response.data.entries;
        this.indexOfLastApi = this.currentPage * this.itemsPerPage;
        this.indexOfFirstApi = this.indexOfLastApi - this.itemsPerPage;
        this.currentApis = this.apis.slice(
          this.indexOfFirstApi,
          this.indexOfLastApi
        );
        for (let i = 1; i <= Math.ceil(this.total / this.itemsPerPage); i++) {
          this.pageNumbers.push(i);
        }
        console.log(this.pageNumbers);
      } catch (error) {
        console.log(error);
      }
    },
    changePage(page) {
      this.currentPage = page;
      this.indexOfLastApi = this.currentPage * this.itemsPerPage;
      this.indexOfFirstApi = this.indexOfLastApi - this.itemsPerPage;
      this.currentApis = this.apis.slice(
        this.indexOfFirstApi,
        this.indexOfLastApi
      );
    },
    changeItemsPerPage() {
      this.indexOfLastApi = this.currentPage * this.itemsPerPage;
      this.indexOfFirstApi = this.indexOfLastApi - this.itemsPerPage;
      this.currentApis = this.apis.slice(
        this.indexOfFirstApi,
        this.indexOfLastApi
      );
    },
  },
  computed: {
    filterApis() {
      if (this.search) {
        return this.currentApis.filter((api) => {
          return this.search
            .toLowerCase()
            .split(" ")
            .every((v) => api.API.toLowerCase().includes(v));
        });
      } else {
        return this.currentApis;
      }
    },
  },
  watch: {
    search() {
      console.log(this.itemsPerPage);
      this.indexOfLastApi = this.currentPage * this.itemsPerPage;
      this.indexOfFirstApi = this.indexOfLastApi - this.itemsPerPage;
      this.currentApis = this.apis.slice(
        this.indexOfFirstApi,
        this.indexOfLastApi
      );
    },
    itemsPerPage() {
      console.log(this.itemsPerPage)
    }
  },
};
</script>

<style></style>
