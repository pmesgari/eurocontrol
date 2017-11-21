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
        v-model="selected"
        item-key="airspaceVersion"
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
        <td class="text-xs-right">{{ props.item.airspaceTimestamp }}</td>
        <td class="text-xs-right">{{ props.item.contourId }}</td>
        <td class="text-xs-right">{{ props.item.airspaceVersion.slice(0,5) }}</td>
      </template>
      <template slot="pageText" slot-scope="{ pageStart, pageStop }">
        {{ pageStart }} to {{ pageStop }}
      </template>
    </v-data-table>
    <v-card-actions class="contour-table-btn table-btn">
      <v-btn class="primary" @click="emitGetVersionedCoordinate">Get Coordinates</v-btn>
      <!-- <v-btn class="warning" @click="emitClearVersions">Clear</v-btn> -->
    </v-card-actions>
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
        selected: [],
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
      emitGetVersionedCoordinate: function (event) {
        this.$emit('getversionedcoordinate', this.selected)
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