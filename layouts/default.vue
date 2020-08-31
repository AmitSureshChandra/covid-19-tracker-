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

        <v-btn text fab @click="theme = !theme; $vuetify.theme.dark = theme">
          <v-icon>{{ theme ? 'mdi-lightbulb-on' : 'mdi-lightbulb-on-outline' }}</v-icon>
        </v-btn>

      </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    <v-row class="px-5">
      <span>data sourced from <a href="https://github.com/CSSEGISandData/COVID-19" target="_blank">CSSEGISandData</a>
      </span>
    </v-row>
    </v-main>
    <v-footer
      app
    >
      <v-row align="right">
        <v-col class="float-right" col="11">
            <span class="float-right">
                - Developed By <span > &nbsp;<b>Amit Kesarwani </b></span>
            </span>
        </v-col>
        <v-col cols="1">
          <v-img src="favicon.ico" width="20" />
        </v-col>

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

      title: 'Covid-19 Tracker',
      theme : false,
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
