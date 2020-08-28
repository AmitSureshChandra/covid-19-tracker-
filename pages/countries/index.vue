<template>
  <div>
     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker Country Wice</h1>

    <v-combobox :items="countries" dense solo color="primary" placeholder="Country Name" v-model="country" />
    <Covid-Card :confirmed="123211" :deaths="313123" :recovered="2132121"/>
  </div>
</template>

<script>
  import CovidCard from "../../components/CovidCard";

  export default {
      components: {CovidCard},
      data(){
        return{
          countries  :[
            'India',
            'America'
          ],
          country : "",

        }
      },
      async mounted(){
        await this.$axios.get('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/08-27-2020.csv')
          .then((res) => {
             let records = res.data.split('\n')
              records.forEach(v => {
                  let data = v.split(',')
                console.log(data)
              })
          })
      }
    }
</script>

