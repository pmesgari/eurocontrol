<template>
  <v-card>
    <v-card-title id="contours-table-header" class="blue white--text darken-1">
      Contours
      <v-spacer></v-spacer>
      <v-text-field
        class="blue white--text darken-1"
        append-icon="search"
        label="Search"
        single-line
        hide-details
        v-model="search"
      ></v-text-field>
    </v-card-title>
    <v-data-table
        v-bind:headers="headers"
        v-bind:items="items"
        v-bind:search="search"
      >
      <template slot="items" slot-scope="props">
        <td @click="emitGetCoordinate" class="text-xs-right">{{ props.item.contourId }}</td>
        <td class="text-xs-right">{{ props.item.longDesc }}</td>
        <td class="text-xs-right">{{ props.item.cwpId }}</td>
      </template>
      <template slot="pageText" slot-scope="{ pageStart, pageStop }">
        {{ pageStart }} to {{ pageStop }}
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
  /* eslint-disable */
  import axios from 'axios'
  export default {
    data () {
      return {
        max25chars: (v) => v.length <= 25 || 'Input too long!',
        tmp: '',
        search: '',
        pagination: {},
        headers: [
          { text: 'Contour ID', value: 'contourId' },
          { text: 'Long Desc', value: 'longDesc' },
          { text: 'CWP ID', value: 'cwpId' }
        ],
        items: [

        ]
      }
    },
    methods: {
      getContours: function () {
        axios({
          method: 'get',
          url: 'http://localhost:8090/airspace/contours',
          auth: {
            username: 'EURO',
            password: 'EURO'
          }
        })
          .then(function (response) {
            console.log(response)
            this.items = response.data
          }.bind(this))
          .catch(function (error) {
            console.log(error)
          })
      },
      emitGetCoordinate: function (event) {
        this.$emit('getcoordinate', event)
      }
    },
    mounted () {
      this.getContours()
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  /* #contours-table-header {
    color:
  } */
</style>