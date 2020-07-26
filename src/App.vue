<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
    >
    <h2 style="color: white">
      DebtGes
    </h2>
    <v-btn-toggle
      group
      mandatory
    >
      <v-btn @click="searchListPicked(0)" outlined style="color: white">
        Search by name
      </v-btn>

      <v-btn @click="searchListPicked(1)"  outlined style="color: white">
        Search by foods
      </v-btn>

      <v-btn @click="searchListPicked(2)" outlined style="color: white">
        Search by brewed date
      </v-btn>

      </v-btn-toggle>

    </v-app-bar>

    <v-main>
      <SearchList v-if="searchPicked === 0" title="Seach for a beer by its name" type="name" placeholder="Search here by name"/>
      <SearchList v-if="searchPicked === 1" title="Search for all beers that match a certain food" type="foods" placeholder='Search here by foods'/>
      <SearchList v-if="searchPicked === 2" title="Search for all beers brewed before a certain date" type="date" placeholder="Search here by brewed date"/>
    </v-main>
  </v-app>
</template>

<script>
import SearchList from './components/SearchList'

export default {
  name: 'App',
  data: () => ({
    searchPicked: 0 // 0 -> Name, 1->Foods, 2->Brewed Date
  }),

  components: {
    SearchList
  },

  methods: {
    searchListPicked (i) {
      this.searchPicked = i
    }
  }
}
</script>
