<template>
  <v-content>
    <v-card>
      <v-card-title class="title">
        <p>{{totalElements}} elements</p>
        <v-spacer></v-spacer>
        <v-text-field v-model="search"
                      append-icon="search"
                      label="Search"
                      single-line
                      hide-details></v-text-field>
      </v-card-title>
      <v-card>
        <v-card-actions class="pa-4">
          <v-flex xs10 sm5 d-flex>
            <v-select
                    :items="risks"
                    item-text="name"
                    item-value="name"
                    label="Risks"
                    :clearable="true"
                    v-model="riskSelected"
            ></v-select>
          </v-flex>
          <v-flex xs10 sm5 d-flex class="ml-3">
            <v-select
                    :items="currencies"
                    :clearable="true"
                    item-text="name"
                    item-value="name"
                    label="Currencies"
                    v-model="currencySelected"
            ></v-select>
          </v-flex>
          <v-flex xs2 sm2 d-flex class="ml-3">
              <v-btn flat small v-on:click="changeFilter()">Apply filter</v-btn>
          </v-flex>
        </v-card-actions>

      </v-card>
      <v-data-table :loading="loading"
                    :headers="headers"
                    :items="list"
                    :pagination.sync="pagination"
                    :search="search"
                    :must-sort="true"
                    item-key="email"
                    v-model="selected"
                    class="elevation-1">

        <v-progress-linear
                slot="progress"
                indeterminate></v-progress-linear>

        <template v-if="show" slot="headerCell"
                  slot-scope="props">
          <v-tooltip bottom>
            <span slot="activator">
              {{ props.header.text }}
            </span>
            <span>
              {{ props.header.text }}
            </span>
          </v-tooltip>
        </template>

        <template v-if="show" slot="items"
                  slot-scope="props">
          <tr :active="props.selected" v-on:click="goTo(props.item)">
            <td class="text-xs">
              <span>{{ props.item.name}}</span>
            </td>
            <td class="text-xs-left">{{ props.item.currency }}</td>
            <td class="text-xs-left">{{ props.item.risk_family | capitalize }}</td>
          </tr>
        </template>

        <v-alert slot="no-results"
                 :value="true"

                 icon="warning">
          Your search for "{{ search }}" found no results.
        </v-alert>

      </v-data-table>

    </v-card>

  </v-content>
</template>

<script>
import baseService from "@/services/base-service.js";

export default {
  data() {
    return {
      show: false,
      search: "",
      riskSelected: null,
      currencySelected: null,
      appFilter: true,
      selected: [],
      pagination: {
        sortBy: "name",
        descending: true,
        rowsPerPage: 100
      },
      headers: [
        {
          text: "Name",
          value: "name"
        },
        {
          text: "Currency",
          align: "left",
          value: "currency"
        },
        {
          text: "Risk",
          align: "left",
          value: "risk_family"
        },
      ]
    };
  },
  mounted: function() {
    this.show = false;
    this.$nextTick(function() {
      this.show = true;
    });
    if (this.list.length === 0) {
      this.$store.dispatch("getListData");
    }
    },
  computed: {
    list() {
      return this.$store.getters.FILTERED_LIST;
    },
    totalElements() {
      return this.$store.getters.FILTERED_LIST.length;
    },
    loading() {
      return this.$store.getters.LIST_UI.loading;
    },
    risks() {
      return this.$store.getters.RISKS;
    },
    currencies() {
      return this.$store.getters.CURRENCIES;
    }
  },
  methods: {
    filterActive() {
      if (this.appFilter === true){
        // filter list payload structure
        const payload = {
          target: 'requests',
          value: 'has_app',
          equal: 'app_active'
        };
        this.$store.commit('filterList', payload);
      } else {
        this.$store.commit('setRequestsData');
      }
      this.appFilter = !this.appFilter;
    },
    goTo(data) {
      this.$router.push({
        name: "element",
        params: {
          id: data.id
        }
      });
    },
    changeFilter(){
      const categories = {
        risk: this.riskSelected,
        currency: this.currencySelected
      };
      console.log(categories);
      this.$store.commit('filterListCat', categories);
    }
  }
};
</script>

<style lang="scss" scoped>
@import "@/assets/sass/module.scss";

</style>
