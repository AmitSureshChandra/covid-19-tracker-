<template>
  <v-card class="pa-6" :loading="loading">

    <div class="float-right">
            <v-btn @click="loadCountryData()" color="error" fab dark>
              <v-icon>mdi-reload</v-icon>
            </v-btn>
    </div>
     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker Country Wice</h1>

    <v-combobox @change="loadCountryData()" :items="countries" dense solo color="primary" placeholder="Country Name" v-model="country" />
    <Covid-Card :confirmed="parseInt(confirmed)" :deaths="parseInt(deaths)" :recovered="parseInt(recovered)"/>
    <div id="pieChartCountry"></div>
    <div align="center">
          <v-chip color="primary" align="center" class="white--text"> Country : {{country}}</v-chip>
    </div>
<!--    Loader-->
<!--    <v-text-field color="success" loading disabled></v-text-field>-->

<!--    Work in progress-->
  </v-card>
</template>

<script>
  import CovidCard from "../../components/CovidCard";
  import moment from 'moment'
  import {GlobalData} from '../../plugins/initializer'
  import c3 from 'c3'

  export default {
      components: {CovidCard},
      data(){
        return{
            data : {},
            countries  :[],
            country : 'India',
            confirmed : 0,
            deaths : 0,
            recovered : 0,

            loading : true,
            pieChartData : [],
            states : [ 'Confirmed','Deaths','Recovered'],
            state : 'Confirmed',
            colors : [
              'orange','red','green'
            ],

        }
      },
      mounted(){
          this.load()
      },
      methods : {
          async load(){

          this.loading = true
          await this.$axios.get('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/'+ moment().subtract(1,'days').format('MM-DD-YYYY') +'.csv')
          .then((res) => {


            let countries = {}
            this.confirmed = 0
            this.deaths = 0
            this.recovered = 0

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

            this.data= countries

            this.loadCountryData()

          })
          .catch(_ => {
            console.log(_)
          })
          .finally(_ => {
            this.loading = false
          })
        },
        loadCountryData(){
            if (this.data){
              let country_ = this.data[this.country]
              this.confirmed = country_.confirmed
              this.deaths = country_.deaths
              this.recovered = country_.recovered
            }

            this.loadChart()
        },
        loadChart(){
             this.countries = Object.keys(this.data)
            this.pieChartData =[
              ['Confirmed' , this.confirmed],
              ['Deaths' , this.deaths],
              ['Recovered' , this.recovered],
            ]

            c3.generate({
                bindto: '#pieChartCountry',
                data: {
                    columns: this.pieChartData,
                    type : 'pie',
                },
                tooltip: {
                  format: {
                    value: function (value, ratio, id, index) { return value; }
                  }
                },
                // pie : {
                //   label: {
                //   format: function (value, ratio, id) {
                //     return value;
                //   }
                // }
                // }
            });
        }
      },
      head(){
        return{
          title : this.country
        }
      }
    }
</script>

