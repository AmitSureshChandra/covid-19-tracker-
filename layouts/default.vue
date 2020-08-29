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
        <v-spacer></v-spacer>

      </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>

    </v-main>
    <v-footer
      app
    >
      <span>&copy; {{ new Date().getFullYear() }} -
          data sourced from <a href="https://github.com/CSSEGISandData/COVID-19" target="_blank">CSSEGISandData</a>
      </span>
      <v-row align="right">
        <v-spacer></v-spacer>
        - Developed By <span > &nbsp;<b>Amit Kesarwani</b></span>
      </v-row>
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
