<template>
    <v-row align="center">
        <v-col cols="12" sm="12" md="4" align="center">
            <v-card color="orange lighten" :loading="loading">
                <v-card-title class="white--text heading">
                    <v-col>Confirmed</v-col>
                    <v-col>{{ confirmed }}</v-col>
                </v-card-title>
            </v-card>
        </v-col>

        <v-col cols="12" sm="12" md="4" align="center">
            <v-card color="red lighten" :loading="loading">
                <v-card-title class="white--text heading">
                    <v-col>Deaths</v-col>
                    <v-col>{{ deaths }}</v-col>
                </v-card-title>
            </v-card>
        </v-col>

        <v-col cols="12" sm="12" md="4" align="center">
            <v-card class="success" :loading="loading">
                <v-card-title class="white--text heading">
                    <v-col>Recovered</v-col>
                    <v-col>{{ recovered }}</v-col>
                </v-card-title>
            </v-card>
        </v-col>
    </v-row>
</template>
<script>
    export default {
      name: 'Covid-Card',
      props: {
        recovered: Number,
        deaths: Number,
        confirmed: Number,
        loading : false,
      }

    }
</script>
