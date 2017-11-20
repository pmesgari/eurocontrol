<template>
  <v-card>
    <v-card-title class="blue lighten-1 white--text">
      Contour Version
      <v-spacer></v-spacer>
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
    name: 'contour-version-table',
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
      // this.getContours()
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  /* td {
    cursor: pointer;
  }
  table.table tbody td, table.table tbody th {
    height: 38px !important;
    font-size: 12px;
    padding: 0 20px;
  } */
</style>