<template>
  <div>
     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker</h1>
  <Covid-Card :confirmed="confirmed" :deaths="deaths" :recovered="recovered" :loading="loading"/>
  </div>
</template>

<script>

  class GlobalData{
      confirmed = 0
      deaths = 0
      recovered = 0
  }

  import CovidCard from "../../components/CovidCard";

  export default {
      components: {CovidCard},
      async mounted() {
        await this.$axios.get('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/08-27-2020.csv')
          .then((res) => {
            this.loading = true

            let countries = {}
            let records = res.data.split('\n')
            records.splice(0,1)

            records.splice(records.length-1,1)
            records.forEach(v => {
              let data = v.split(',')

              if (Object.keys(countries).includes(data[3])){
                    let country = countries[data[3]]
                    country.country = data[3]
                    country.confirmed =  parseInt(country.confirmed) + parseInt(data[7])
                    country.deaths =  parseInt(country.deaths) + parseInt(data[8])
                    country.recovered =  parseInt(country.recovered) + parseInt(data[9])

              }else{
                let country = new GlobalData()
                country.country = data[3]
                country.confirmed += data[7]
                country.deaths += data[8]
                country.recovered += data[9]
                countries[data[3]] = country
              }
            })
            //records ready


            Object.values(countries).forEach(v => {
                this.confirmed = parseInt(this.confirmed) + parseInt(v.confirmed)
                this.deaths = parseInt(this.deaths) + parseInt(v.deaths)
                this.recovered = parseInt(this.recovered) + parseInt(v.recovered)
            })

          })
          .catch(_ => {
            console.log('failed to load data')
          })
          .finally(_ => {
            this.loading = false
          })
      },
      data(){
          return{
            confirmed : 0,
            deaths : 0,
            recovered : 0,
            loading : true,
          }
      }
  }
</script>

