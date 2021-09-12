<template>
  <v-app>
    <v-app-bar
      app
      color="#6A1B9A"
      dark
    >
      <div class="d-flex align-center">
        <h3>Fuel Stations - autoSense</h3>
      </div>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>
