<template>
  <v-app id="inspire">
    <v-toolbar color="indigo" dark fixed app>
      <v-toolbar-title>Airspace Validation</v-toolbar-title>
    </v-toolbar>
    <v-content>
      <airspace></airspace>
    </v-content>
    <v-footer color="indigo" app>
      <span class="white--text">&copy; 2017</span>
    </v-footer>
  </v-app>
</template>

<script>
  /* eslint-disable */
  import Airspace from './components/Airspace'
  
  export default {
    components: {'airspace': Airspace},
    data () {
      return {
      }
    }
  }
</script>