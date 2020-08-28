<template>
  <div>
<!--     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker Country Wice</h1>-->

<!--    <v-combobox :items="countries" dense solo color="primary" placeholder="Country Name" v-model="country" />-->
<!--    <Covid-Card :confirmed="123211" :deaths="313123" :recovered="2132121"/>-->

    <v-text-field color="success" loading disabled></v-text-field>

    Work in progress
  </div>
</template>

<script>
  import CovidCard from "../../components/CovidCard";

  export default {
      components: {CovidCard},
      data(){
        return{
          countries  :[

          ],
          country : "",

        }
      },
      async mounted(){

      }
    }
</script>

