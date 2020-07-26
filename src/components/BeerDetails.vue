<template>
    <v-card>
        <v-card-title>
          {{ beer.name }}
        </v-card-title>
        <v-card-subtitle class="mb-n8">
          <v-container>
            <v-row align="center">
              <v-col xs="12" sm="2" md="2">
                <v-img max-height="70" contain :src=beer.img />
              </v-col>
              <v-col xs="12" md="10" justify="center">
                <v-col xs="12" sm="12" md="12" class="d-flex justify-start" >
                  {{ beer.tagline }}
                </v-col>
                <v-col xs="12" sm="12" md="12" class="d-flex justify-start">
                  <span class="caption"><b>Contributed by:</b> {{ beer.contributed_by}}</span>
                </v-col>
              </v-col>
            </v-row>
          </v-container>
        </v-card-subtitle>
        <hr/>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col xs="12" sm="12" md="12">
                <b>Bewer tips:</b> {{ beer.brewers_tips }}
              </v-col>
              <v-col xs="12" sm="12" md="12">
                <b>Food pairing: </b>
                <ul>
                  <li v-for="food in beer.food_pairing" v-bind:key="food">
                    {{ food }}
                  </li>
                </ul>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
    </v-card>
</template>

<script>
export default {
  name: 'BeerDetails',
  props: {
    beer: Object
  }
}
</script>
