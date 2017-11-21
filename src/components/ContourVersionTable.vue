<template>
  <v-card>
    <v-card-title class="blue lighten-1 white--text">
      Contour Version
      <v-spacer></v-spacer>
    </v-card-title>
    <v-data-table
        v-bind:headers="headers"
        v-bind:items="this.airspaceItems"
        v-bind:search="search"
      >
      <template slot="items" slot-scope="props">
        <td class="text-xs-right">{{ props.item.airspaceTimestamp }}</td>
        <td @click="emitGetCoordinate" class="text-xs-right">{{ props.item.contourId }}</td>
        <td class="text-xs-right">{{ props.item.airspaceVersion }}</td>
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
    props: ['airspaceVersions'],
    data () {
      return {
        max25chars: (v) => v.length <= 25 || 'Input too long!',
        tmp: '',
        search: '',
        pagination: {},
        headers: [
          { text: 'Timestamp', value: 'airspaceTimestamp' },
          { text: 'ID', value: 'contourId' },
          { text: 'Version', value: 'airspaceVersion', sortable: false}
        ],
        items: [
        ]
      }
    },
    methods: {
      emitGetCoordinate: function (event) {
        this.$emit('getcoordinate', this.selected)
      }
    },
    mounted () {
    },
    computed: {
      airspaceItems: function () {
        return this.airspaceVersions
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

</style>