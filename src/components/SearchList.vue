<template>
  <v-container>
    <v-row>
      <v-col xs="12">
        <h1>{{ title }}</h1>
      </v-col>
    </v-row>
    <v-row :id="type" v-if="type != 'date'">
      <v-col cols="4">
        <v-text-field
          :placeholder=placeholder
          v-model="inputText" />
      </v-col>
      <v-col align-self="center" cols="2">
        <v-btn @click="onClick(true)" color="primary">Search by {{ type }} </v-btn>
      </v-col>
    </v-row>

    <v-row v-if="type === 'date'">
      <v-col sm="5" md="3" lg="3">
        <Datepicker style="text-align-last: center" format="MM-yyyy" minimum-view='month' maximum-view='month'
        @selected="selectedDate" :value="date" id="datepicker" :bootstrap-styling="true">
        </Datepicker>
      </v-col>
      <v-col sm="7" md="4" lg="4">
        <v-btn @click="onClick(true)" color="primary">Search by date</v-btn>
      </v-col>
    </v-row>

    <v-row>
        <v-col class="text-sm-end text-md-end" xs="12" sm="6" md="6">
          <v-btn class="black--text" @click="onClickPrevious()" :disabled="previousButtonDisable" color="grey lighten-3">
            <v-icon class="pr-2">mdi-arrow-left</v-icon>
            Previous
          </v-btn>
        </v-col>
        <v-col class="text-sm-star text-md-start" xs="12" sm="6" md="6">
          <v-btn class="black--text" @click="onClick(false)" :disabled="nextButtonDisable" color="grey lighten-3">
            Next
            <v-icon class="pl-2">mdi-arrow-right</v-icon>
          </v-btn>
        </v-col>
    </v-row>

    <v-row>
      <v-col class="text-center" md="12">
        {{ currentPage > 0 ? "Page: " + currentPage : '' }}
      </v-col>
    </v-row>

    <v-row v-if="notFound">
      <v-col  class="text-center" md="12">
        No results found
      </v-col>
    </v-row>

    <v-row v-for="beer in beers" v-bind:key="beer.id">
      <v-col>
        <v-card elevation=5>
          <v-card-title>
            {{ beer.name }}
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row align="center">
                <v-col xs="12" sm="2" md="2">
                  <v-img max-height="70" contain :src=beer.img />
                </v-col>
                <v-col xs="12" sm="8" md="8">
                  {{ beer.description }}
                </v-col>
                <v-col xs="12" sm="2" md="2" class="text-end">
                  <v-btn :id="beer.id" @click.stop="dialog = true; beerDetails = beer" color="primary">
                    <v-icon class="pr-2">mdi-information-outline</v-icon>
                    Details
                  </v-btn>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-dialog v-model="dialog" width="600px">
      <BeerDetails :beer="beerDetails" />
    </v-dialog>

  </v-container>
</template>

<script>

import Datepicker from 'vuejs-datepicker'
import BeerDetails from './BeerDetails'

export default {
  name: 'SearchList',
  components: {
    Datepicker,
    BeerDetails
  },
  props: {
    title: String,
    type: String,
    placeholder: String
  },
  watch: {
    type: function (val, oldVal) {
      this.inputText = ''
      this.date = new Date()
      this.beers = []
      this.beerDetails = Object
      this.beersTotal = new Map()
      this.nextButtonDisable = true
      this.previousButtonDisable = true
      this.currentPage = 0
      this.notFound = false
    }
  },
  data: function () {
    return {
      inputText: '',
      date: new Date(),
      beers: [],
      beersTotal: new Map(),
      nextButtonDisable: true,
      previousButtonDisable: true,
      currentPage: 0,
      dialog: false,
      beerDetails: {},
      notFound: false
    }
  },
  methods: {
    selectedDate: function (e) {
      this.date = e
    },
    onClick: function (isSearch) {
      if (isSearch) {
        this.currentPage = 1
        this.beers = []
        this.beersTotal = new Map()
      } else {
        this.previousButtonDisable = false
        this.currentPage++
      }
      if (this.beersTotal.has(this.currentPage)) {
        // quando já tem guardada as beers no Map não precisa de fazer o fetch à api
        this.beers = this.beersTotal.get(this.currentPage).slice()
        if (this.beersTotal.has(this.currentPage + 1)) {
          this.nextButtonDisable = false
        } else {
          this.nextButtonDisable = true
        }
      } else {
        // quando ainda não tem guardada as beers no Map precisa de fazer o fetch à api, em seguida guarda no Map para as
        // leituras seguintes não precisarem de fazer o fetch à api
        var vm = this

        var url = 'https://api.punkapi.com/v2/beers'
        if (vm.type === 'name') {
          url += '?beer_name=' + this.inputText.split(' ').join('_') + '&page=' + vm.currentPage
        } else if (vm.type === 'foods') {
          url += '?food=' + this.inputText.split(' ').join('_') + '&page=' + vm.currentPage
        } else {
          var month = vm.date.getMonth() + 1
          url += '?brewed_before=' + month + '-' + vm.date.getFullYear() + '&page=' + vm.currentPage
        }
        console.log(url)
        this.nextButtonDisable = true
        fetch(url)
          .then(function (response) {
            return response.json()
          })
          .then(function (json) {
            vm.nextButtonDisable = false
            if (json.length === 25) {
              vm.nextButtonDisable = false
            } else if (json.length < 25) {
              vm.nextButtonDisable = true
            }
            if (json.length > 0) {
              vm.notFound = false
              vm.beers = []
              for (var i = 0; i < json.length; i++) {
                var beer = {
                  name: json[i].name,
                  id: json[i].id,
                  tagline: json[i].tagline,
                  description: json[i].description,
                  img: json[i].image_url,
                  food_pairing: json[i].food_pairing,
                  brewers_tips: json[i].brewers_tips,
                  contributed_by: json[i].contributed_by
                }
                vm.beers.push(beer)
              }
              vm.beersTotal.set(vm.currentPage, vm.beers)
            } else {
              vm.notFound = true
            }
          })
      }
    },
    onClickPrevious: function () {
      if (this.beersTotal.size > 1) {
        this.nextButtonDisable = false
      }

      if (this.currentPage === 2) {
        this.previousButtonDisable = true
      } else {
        this.previousButtonDisable = false
      }
      this.beers = []
      this.currentPage--
      this.beers = this.beersTotal.get(this.currentPage).slice()
    }
  }
}
</script>

<style>
#datepicker {
  background: #f2f2f2;
  border: 1px solid #ddd;
  padding: 0em 1em 1em;
  margin-bottom: 2em;
}

</style>
