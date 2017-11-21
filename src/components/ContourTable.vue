<template>
  <v-card>
    <v-card-title id="contours-table-header" class="blue white--text darken-2">
      Contours
      <v-spacer></v-spacer>
      <v-text-field
        class="blue white--text darken-2"
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
        v-model="selected"
        item-key="contourId"
        select-all
        class="elevation-1"
      >
      <template slot="items" slot-scope="props">
        <td>
          <v-checkbox
            primary
            hide-details
            v-model="props.selected"
          ></v-checkbox>
        </td>
        <td class="text-xs-right">{{ props.item.contourId }}</td>
        <td class="text-xs-right">{{ props.item.cwpId }}</td>
      </template>
      <template slot="pageText" slot-scope="{ pageStart, pageStop }">
        {{ pageStart }} to {{ pageStop }}
      </template>
    </v-data-table>
    <v-card-actions class="contour-table-btn table-btn">
      <v-btn class="primary" @click="emitGetCoordinate">Get Coordinates</v-btn>
      <v-btn class="primary" @click="emitGetVersions">Get Versions</v-btn>
      <v-btn class="warning" @click="emitClearVersions">Clear</v-btn>
    </v-card-actions>
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
          { text: 'CWP ID', value: 'cwpId' }
        ],
        items: [],
        selected: []
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
      emitGetVersions: function (event) {
        this.$emit('getversions', this.selected)
      },
      emitClearVersions: function (event) {
        this.selected = []
        this.$emit('clearversions', event)
      },
      emitGetCoordinate: function (event) {
        this.$emit('getcoordinate', this.selected)
      }
    },
    mounted () {
      this.getContours()
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .table-btn {
    justify-content: center;
  }
</style>