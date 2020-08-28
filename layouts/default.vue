<template>
  <v-app dark>
    <v-app-bar
      fixed
      app
      i
      max-height="70px"
    >
      <v-app-bar-nav-icon v-show="false" />

        <nuxt-link to="/">Home</nuxt-link>
        <nuxt-link to="/countries">Countries</nuxt-link>

      </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>
    <v-footer
      app
    >
      <span>&copy; {{ new Date().getFullYear() }}  covid-19 tracker</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        }
      ],

      title: 'Covid-19 Tracker'
    }
  }
}
</script>
<style>
  a{
    text-decoration: none;
    font-weight: bold;
    margin-left: 10px;
    margin-right: 20px;
  }
  .title {
    margin-right: 30px;
    font-size: 40px;
  }
</style>
