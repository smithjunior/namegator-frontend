<template>
  <div>
    <div class="container">
      <div class="row">
        <div class="col-md">
          <AppItemList
            title="Prefixos"
            v-bind:items="prefixes"
            v-on:addItem="addPrefix"
            v-on:deleteItem="deletePrefix"
          ></AppItemList>
        </div>
        <div class="col-md">
          <AppItemList
            title="Sufixos"
            v-bind:items="sufixes"
            v-on:addItem="addSufix"
            v-on:deleteItem="deleteSufix"
          ></AppItemList>
        </div>
      </div>
      <br />
      <h5>
        Dominios
        <span class="badge badge-info">{{ domains.length }}</span>
      </h5>
      <br />
      <div class="card">
        <div class="card-body">
          <ul class="group-list">
            <li class="list-group-item" v-for="domain in domains" v-bind:key="domain.name">
              <div class="row">
                <div class="col-md">{{ domain.name }}</div>
                <div class="col-md text-right">
                  <a class="btn btn-info" v-bind:href="domain.checkout" target="_black">
                    <span class="fa fa-shopping-cart"></span>
                  </a>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios/dist/axios";
import AppItemList from "./AppItemList";

export default {
  name: "DomainList",
  components: {
    AppItemList
  },
  data: function() {
    return {
      prefixes: [],
      sufixes: []
    };
  },
  methods: {
    addPrefix(prefix) {
      this.prefixes.push(prefix);
      this.prefix = "";
    },
    addSufix(sufix) {
      this.sufixes.push(sufix);
      this.sufix = "";
    },
    deletePrefix(prefix) {
      this.prefixes.splice(this.prefixes.indexOf(prefix), 1);
    },
    deleteSufix(sufix) {
      this.sufixes.splice(this.sufixes.indexOf(sufix), 1);
    }
  },
  computed: {
    domains() {
      const domains = [];
      for (const prefix of this.prefixes) {
        for (const sufix of this.sufixes) {
          const name = prefix + sufix;
          const url = name.toLowerCase();
          const checkout = `https://checkout.hostgator.com.br/?a=add&sld=${url}&tld=.com.br`;
          domains.push({ name, checkout });
        }
      }
      return domains;
    }
  },
  created() {
    axios({
      url: process.env.VUE_APP_APOLLO_ENDPOINT_URL,
      method: "post",
      data: {
        query: `
        {
          prefixes: items (type: "prefix") {
            id, type, description
          },
          sufixes: items (type: "sufix") {
            id, type, description
          }
        }
        `
      }
    }).then(response => {
      const query = response.data;
      this.prefixes = query.data.prefixes.map(prefix => prefix.description);
      this.sufixes = query.data.sufixes.map(sufix => sufix.description);
    });
  }
};
</script>

<style>
</style>
