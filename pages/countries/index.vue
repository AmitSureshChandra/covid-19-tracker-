<template>
  <v-card class="pa-6" :loading="loading">

    <div class="float-right">

    </div>
    <h4>
      <v-row>
        <v-col cols="7" sm="4">
          <v-tooltip right>
              <template v-slot:activator="{ on, attrs }" >
                <v-menu
                  v-bind="attrs"
                  v-on="on"
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
              </template>

            <h5 > Choose Date Uptill you want to see Data</h5>
          </v-tooltip>

        </v-col>

        <v-col cols="5" sm="8">
          <v-btn @click="loadCountryData()" class="float-right" color="error" fab dark>
            <v-icon>mdi-reload</v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </h4>
     <h1 class="pa-4 display-2 bold my-3" align="center">Covid-19 Tracker Country Wice</h1>

    <v-combobox @change="loadCountryData()" :items="countries" dense solo color="primary" placeholder="Country Name" v-model="country" >
      <template v-slot:no-data>
          <span class="px-4">No Country Available</span>
      </template>
    </v-combobox>

    <Covid-Card :confirmed="parseInt(confirmed)" :deaths="parseInt(deaths)" :recovered="parseInt(recovered)"/>

    <v-combobox class="mt-4" @change="loadStateData()" :items="country_states" dense solo color="primary" placeholder="State Name" v-model="country_state" >
      <template v-slot:no-data>
        <span class="px-4">No State/Area Available</span>
      </template>
    </v-combobox>

    <Covid-Card :confirmed="parseInt(state_confirmed)" :deaths="parseInt(state_deaths)" :recovered="parseInt(state_recovered)"/>

    <v-row>
      <v-col cols="12" sm="6">
        <div id="pieChartCountry"></div>

        <div align="center" >
          <v-chip color="primary" align="center" class="white--text">  Country : {{ country }}</v-chip>
        </div>
      </v-col>

      <v-col cols="12" sm="6" >
        <div id="pieChartState"></div>

        <div align="center" >
          <v-chip color="primary" align="center" class="white--text"> State : {{ country_state }}</v-chip>
        </div>
      </v-col>
    </v-row>

<!--    <v-row>-->
<!--      <div id="linechart"></div>-->
<!--    </v-row>-->

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
            confirmed : 'NA',
            deaths : 'NA',
            recovered : 'NA',

            loading : true,
            pieChartData : [],
            pieChartStateData : [],

            states : [ 'Confirmed','Deaths','Recovered'],
            state : 'Confirmed',
            colors : [
              'orange','red','green'
            ],
            date : new Date().toISOString().substr(0, 10),
            theme : false,
            menu : false,
            country_states : [],
            country_state : "",
            state_confirmed : 'NA',
            state_deaths : 'NA',
            state_recovered : 'NA',
        }
      },
      watch : {
        date(v){
          this.load()
        }
      },
      mounted(){
          this.load()
      },
      computed : {
        getComputedDate(){
          return this.$moment(this.date).format('MM-DD-YYYY')
        },

        // for showing to user
        getComputedDate2(){
          return this.$moment(this.date).format('DD-MM-YYYY')
        },
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

            this.loading = true
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

                    if (data[2]){
                      country.area.push(data[2])
                      country.area_confirm.push(data[7])
                      country.area_death.push(data[8])
                      country.area_recover.push(data[9])
                    }

              }else{
                let country = new GlobalData()

                country.country = data[3]
                country.confirmed += data[7]
                country.deaths += data[8]
                country.recovered += data[9]

                if (data[2]){
                  country.area.push(data[2])
                  country.area_confirm.push(data[7])
                  country.area_death.push(data[8])
                  country.area_recover.push(data[9])
                }

                countries[data[3]] = country
              }
            })

            this.data= countries
            this.loadCountryData()
          })
          .catch(_ => {
            console.log(_)
            alert('Something Went Wrong ... Please Select Proper Date')
          })
          .finally(_ => {
            this.loading = false
          })
        },
        loadStateData(){

          this.state_confirmed = "NA"
          this.state_deaths = "NA"
          this.state_recovered = "NA"

          if (this.data){

            let country_ = this.data[this.country]

            this.country_states.forEach((v,index) => {
              if (v === this.country_state){
                this.state_confirmed = country_.area_confirm[index]
                this.state_deaths = country_.area_death[index]
                this.state_recovered = country_.area_recover[index]
                this.pieChartStateData = [ this.country_states ,this.state_confirmed, this.state_deaths, this.state_recovered]
              }
            })
          }
        },
        loadCountryData(){
            if (this.data){
                this.country_states = []
                let country_ = this.data[this.country]
                this.confirmed = country_.confirmed
                this.deaths = country_.deaths
                this.recovered = country_.recovered

                if (country_.area && country_.area.length > 0)
                {
                  let index = 0
                  this.country_states = country_.area
                  this.country_state = this.country_states[index]
                  this.state_confirmed = country_.area_confirm[index]
                  this.state_deaths = country_.area_death[index]
                  this.state_recovered = country_.area_recover[index]
                }else{
                  this.country_states = country_.area
                  this.state_confirmed = "NA"
                  this.state_deaths = "NA"
                  this.state_recovered = "NA"
                  this.country_state = "Not Available"
                }
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


          this.pieChartStateData =[
            ['Confirmed' , this.state_confirmed],
            ['Deaths' , this.state_deaths],
            ['Recovered' , this.state_recovered],
          ]

          c3.generate({
            bindto: '#pieChartState',
            data: {
              columns: this.pieChartStateData,
              type : 'pie',
            },
            tooltip: {
              format: {
                value: function (value, ratio, id, index) { return value; }
              }
            },

          });

          // c3.generate({
          //   bindto: '#linechart',
          //
          //   data: {
          //     columns: [
          //       ['state', this.state_confirmed, this.state_deaths, this.state_recovered],
          //       ['country', this.confirmed, this.deaths, this.recovered],
          //     ]
          //   },
          // });
        }
      },
      head(){
        return{
          title : this.country
        }
      }
    }
</script>

