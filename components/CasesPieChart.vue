<template>
    <div id="pieCaseChart"></div>
</template>

<script>

  import c3 from "c3"

    export default {
        name: "CasesPieChart",
        props : {
            countries : Object
        },
        watch : {
            countries(value){
                this.data =[]
                Object.values(value).forEach(v => { this.data.push([v.country , parseInt(v.confirmed)])   })
                this.data.splice(3,this.data.length-2)
                console.log(JSON.stringify(this.data))
            }
        },
        mounted() {
            c3.generate({
                  bindto: '#pieCaseChart',
                  data: {
                      columns: this.data,
                      type : 'pie',
                  }
              });

        },
        data(){
          return{
            data : [],
          }
        }
    }
</script>

<style scoped>
  @import 'https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.css';
</style>
