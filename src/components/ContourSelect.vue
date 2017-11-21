<template>
 <v-layout wrap>
    <v-flex xs12>
      <v-select
        v-model="select"
        label="Select a contour"
        chips
        debounce-search
        clearable
        tags
        :items="items"
      ></v-select>
    </v-flex>
    <v-flex d-flex class="justify-xs-center">
      <v-btn class="primary" @click="emitGetCoordinate">Get Coordinates</v-btn>
      <v-btn class="warning" @click="emitClearCoordinates">Clear</v-btn>
    </v-flex>
 </v-layout>
</template>

<script>
  /* eslint-disable */
  import axios from 'axios'
  export default {
    name: 'contour-select',
    data () {
      return {
        select: [],
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
            this.items = response.data.map(d => d.contourId)
          }.bind(this))
          .catch(function (error) {
            console.log(error)
          })
      },
      emitGetCoordinate: function (event) {
        this.$emit('getcoordinate', this.select)
      },
      emitClearCoordinates: function () {
        this.$emit('clearCoordinates')
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