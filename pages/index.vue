<template>
  <v-card class="px-6 pt-4" :loading="loading">

      <v-row>

        <v-col cols="8" sm="4" class="float-left" align="left">
          <h4>
            <v-menu
              ref="menu"
              v-model="menu"
              :close-on-content-click="false"
              :return-value.sync="date"
              transition="scale-transition"
              offset-y
              min-width="290px"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="getComputedDate2"
                  label="Data Uptill"
                  prepend-icon="mdi-calendar"
                  :max="new Date()"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker v-model="date" no-title scrollable>
                <v-spacer></v-spacer>
                <v-btn text color="primary" @click="menu = false">Cancel</v-btn>
                <v-btn text color="primary" @click="$refs.menu.save(date)">OK</v-btn>
              </v-date-picker>
            </v-menu>
          </h4>
        </v-col>
        <v-spacer></v-spacer>
<!--        <date-picker type="date" format="DD-MM-YYYY" v-model="date" valueType="format"></date-picker>-->
        <v-col>
          <v-btn class="float-right" @click="load()" color="error" fab dark>
            <v-icon>mdi-reload</v-icon>
          </v-btn>
        </v-col>
      </v-row>

      <div class="float-right">

      </div>

     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker</h1>
     <Covid-Card :confirmed="confirmed" :deaths="deaths" :recovered="recovered" :loading="loading"/>


    <div id="pieCaseChartConfirmed"></div>
    <div align="center">
          <v-chip :color="colors[getStates()-1]" align="center" class="white--text">Countries having &nbsp; <span class="white--text darken">{{ this.state}} Case</span> &nbsp; > 100000</v-chip>
    </div>

     <div>
        <v-alert>
            <v-row class="ma-3">
            <v-select  outlined :items="states" v-model="state"  label="States"  dense class="ma-3" @change="load" >
            </v-select>

            <v-text-field outlined   v-model="count" label="Countries having Minimum Count" type="number"  dense class="ma-3" @change="load" >
            </v-text-field>
          </v-row>
        </v-alert>
      </div>

  </v-card>
</template>

<script>

  import CovidCard from "../components/CovidCard";
  import c3 from 'c3'
  import moment from 'moment'
  import DatePicker from 'vue2-datepicker';
  import 'vue2-datepicker/index.css';
  import {GlobalData} from "../plugins/initializer";


  export default {
      components: {
        CovidCard,
        DatePicker
      },
      watch : {
          date(v){
            this.load()
          }
      },
      computed:{
          getComputedDate(){
            return this.$moment(this.date).format('MM-DD-YYYY')
          },

        // for showing to user
          getComputedDate2(){
            return this.$moment(this.date).format('DD-MM-YYYY')
          },



      },
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
            ],
            menu  : false,
            date : new Date().toISOString().substr(0, 10),
          }
      },
      methods : {
        async load(){

          this.loading = true
          let prev =2
          if(moment().hour() > 10){
            prev = 1;
          }
          if (this.$moment().diff(this.date,'days') >= 2){
            prev = 0
          }

          await this.$axios.get('https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/'+ this.$moment(this.date).subtract(prev,'days').format('MM-DD-YYYY') +'.csv')
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
            alert('Something went Wrong ... try to choose Correct date')
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

