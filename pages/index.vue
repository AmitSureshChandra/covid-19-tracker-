<template>
  <div>
     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker</h1>
     <Covid-Card :confirmed="confirmed" :deaths="deaths" :recovered="recovered" :loading="loading"/>


    <div id="pieCaseChartConfirmed"></div>
    <div align="center">
          <v-chip :color="colors[getStates()-1]" align="center" class="white--text">Countries having &nbsp; <span class="white--text darken">{{ this.state}} Case</span> &nbsp; > 100000</v-chip>
    </div>

     <div>
        <v-alert>
            <v-row class="ma-3">
            <v-select :items="states" v-model="state"  label="States"  dense class="ma-3" @change="load" >
            </v-select>

            <v-text-field  v-model="count" label="Countries having Minimum Count"  dense class="ma-3" @change="load" >
            </v-text-field>
          </v-row>
        </v-alert>
      </div>

  </div>
</template>

<script>


  class GlobalData{
      confirmed = 0
      deaths = 0
      recovered = 0
  }

  import CovidCard from "../components/CovidCard";
  import c3 from 'c3'
  import moment from 'moment'
  export default {
      components: { CovidCard},
      mounted() {
          this.load()
      },
      data(){
          return{
            confirmed : 0,
            deaths : 0,
            recovered : 0,
            loading : true,
            countries : {},
            pieChartData : [],
            states : [ 'Confirmed','Deaths','Recovered'],
            state : 'Confirmed',
            count :100000,
            colors : [
              'orange','red','green'
            ]
          }
      },
      methods : {
        async load(){
          await this.$axios.get('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/'+ moment().subtract(2,'days').format('MM-DD-YYYY') +'.csv')
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

            this.countries = countries

            this.pieChartData =[]


            Object.values(countries).forEach(v => {
                this.confirmed = parseInt(this.confirmed) + parseInt(v.confirmed)
                this.deaths = parseInt(this.deaths) + parseInt(v.deaths)
                this.recovered = parseInt(this.recovered) + parseInt(v.recovered)
                let check =  this.getStates()
                let val = check === 1 ? parseInt(v.confirmed) : ( check === 2 ? parseInt(v.deaths) : parseInt(v.recovered) )
                if (val > this.count){
                  this.pieChartData.push([v.country , val ])
                }
            })

            c3.generate({
                bindto: '#pieCaseChartConfirmed',

                data: {
                    columns: this.pieChartData,
                    type : 'pie',
                },
                tooltip: {
                  format: {
                    value: function (value, ratio, id, index) { return value; }
                  }
                },
                pie : {
                  label: {
                  format: function (value, ratio, id) {
                    return value;
                  }
                }
                }
            });

          })
          .catch(_ => {
            console.log(_)
          })
          .finally(_ => {
            this.loading = false
          })
        },

        getStates(){
          switch (this.state) {
            case "Confirmed":
                return 1;
            case "Deaths":
                return 2;
            case "Recovered":
                return 3;
          }
        }
      },
    head(){
        return{
            title : 'Covid-19 Tracking'
        }
    }
  }
</script>

<style>
  @import 'https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.css';
</style>

