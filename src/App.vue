<template>
  <v-app>
    <v-navigation-drawer
      persistent
      :clipped="clipped"
      v-model="drawer"
      enable-resize-watcher
      style="z-index: 10000;"
      fixed
      app
    >
      <v-form style="margin: 50px 20px; ">
        <v-text-field
          label="How many frames by move click ?"
          placeholder="Integer"
          v-model="nbFrames"
        ></v-text-field>
        <v-switch v-model="persistence" :label="'Persistence'"></v-switch>
        <v-switch v-model="mapDisplay" :label="'Display map'" @change="sendMapEvent"></v-switch>
      </v-form>
    </v-navigation-drawer>
    <v-toolbar
      app
      :clipped-left="clipped"
    >
      <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      <v-toolbar-title v-text="title"></v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items class="modify-tool">

      </v-toolbar-items>
    </v-toolbar>
    <v-content>
      <Map ref="mapComponent" :nb-frames="nbFrames" :persistence="persistence"></Map>
    </v-content>

    <v-footer :fixed="fixed" app>
    </v-footer>
  </v-app>
</template>

<script>
  import Map from './components/Map'

  export default {
    data () {
      return {
        clipped: true,
        drawer: false,
        fixed: false,
        items: [],
        right: true,
        rightDrawer: false,
        title: 'Fortnite Random Walk',
        nbFrames: 100,
        persistence: true,
        mapDisplay: false
      }
    },
    name: 'App',
    components: {
      Map,
    },
    methods: {
      sendMapEvent() {
        this.$refs.mapComponent.displayMap(this.mapDisplay);
      }
    },
    mounted() {
    }
  }
</script>
