<template>
  <v-card style="text-align: center;">
    <v-card-title class="blue lighten-1 white--text">
      Coordinates
      <v-spacer></v-spacer>
    </v-card-title>
    <v-data-table
        v-bind:headers="headers"
        v-bind:items="this.tableCoordinates"
        v-bind:search="search"
      >
      <template slot="items" slot-scope="props">
        <td class="text-xs-right">
          <v-edit-dialog
            @open="tmp = props.item.lat"
            @save="props.item.lat = tmp || props.item.lat"
            large
            lazy
          >
            <div>{{ props.item.lat }}</div>
            <div slot="input" class="mt-3 title">Update Coordinates</div>
            <v-text-field
              slot="input"
              label="Edit"
              v-model="tmp"
              single-line
              counter
              autofocus
              :rules="[max25chars]"
            ></v-text-field>
          </v-edit-dialog>
        </td>
        <td class="text-xs-right">
          <v-edit-dialog
            @open="tmp = props.item.lng"
            @save="props.item.lng = tmp || props.item.lng"
            large
            lazy
          >
            <div>{{ props.item.lng }}</div>
            <div slot="input" class="mt-3 title">Update Coordinates</div>
            <v-text-field
              slot="input"
              label="Edit"
              v-model="tmp"
              single-line
              counter
              autofocus
              :rules="[max25chars]"
            ></v-text-field>
          </v-edit-dialog>
        </td>
        <td class="text-xs-right">{{ props.item.areaContourId || props.item.airspaceVersion.slice(0,5) }}</td>
      </template>
      <template slot="pageText" slot-scope="{ pageStart, pageStop }">
        {{ pageStart }} to {{ pageStop }}
      </template>
    </v-data-table>
    <v-btn @click="emitDraw" success>Draw</v-btn>
    <v-btn @click="emitValidate" primary>Validate</v-btn>
  </v-card>
</template>

<script>
export default {
  name: 'contour-points-table',
  props: ['coordinates'],
  data () {
    return {
      max25chars: (v) => v.length <= 25 || 'Input too long!',
      tmp: '',
      search: '',
      pagination: {},
      headers: [
        { text: 'Latitude', value: 'lat', sortable: false },
        { text: 'Longitude', value: 'lng', sortable: false },
        { text: 'ID', value: 'areaContourId', sortable: false }
      ],
      items: []
    }
  },
  methods: {
    emitDraw: function () {
      this.$emit('requestDraw', this.items)
    },
    emitValidate: function () {
      this.$emit('requestValidation', this.items)
    }
  },
  computed: {
    tableCoordinates: function () {
      this.items = this.coordinates.map(o => Object.assign({}, o))
      return this.items
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

</style>

