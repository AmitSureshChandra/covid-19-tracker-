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
            confirmed : 'NA',
            deaths : 'NA',
            recovered : 'NA',

            loading : true,
            pieChartData : [],
            states : [ 'Confirmed','Deaths','Recovered'],
            state : 'Confirmed',
            colors : [
              'orange','red','green'
            ],
          date : new Date().toISOString().substr(0, 10),
            theme : false,
            menu : false,
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
            alert('Something Went Wrong ... Please Select Proper Date')
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

