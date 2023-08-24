<template>

  <div>
    <v-row class="mb-6" no-gutters>
    <v-col cols="12" lg="4" xl="3">
      <v-card class="ma-5 pa-5">
        <div>
          <p class="text-h6 ma-0 primary--text">TOTAL LEASING</p>
          <p class="text-body-1 mb-2">Jumlah leasing</p>
          <div v-if="loading" class="text-medium ma-0">
           Loading...
          </div>
          <h1 v-else class="text-h3 ma-0 font-weight-bold">
            {{ addCommas(homeTotal.leasing) }}
          </h1>
        </div>
      </v-card>
    </v-col>
    <v-col cols="12" lg="4" xl="3">
      <v-card class="ma-5 pa-5">
        <div>
          <p class="text-h6 ma-0 primary--text">TOTAL MEMBER TRIAL</p>
          <p class="text-body-1 mb-2">Jumlah member trial</p>
          <div v-if="loading" class="text-medium ma-0">
           Loading...
          </div>
          <h1 v-else class="text-h3 ma-0 font-weight-bold">
            {{ addCommas(homeTotal.trial_members) }}
          </h1>
        </div>
      </v-card>
    </v-col>
    <v-col cols="12" lg="4" xl="3">
      <v-card class="ma-5 pa-5">
        <div>
          <p class="text-h6 ma-0 primary--text">TOTAL MEMBER PREMIUM</p>
          <p class="text-body-1 mb-2">Jumlah member premium</p>
          <div v-if="loading" class="text-medium ma-0">
           Loading...
          </div>
          <h1 v-else class="text-h3 ma-0 font-weight-bold">
            {{ addCommas(homeTotal.premium_members) }}
          </h1>
        </div>
      </v-card>
    </v-col>
    <v-col cols="12" lg="4" xl="3">
      <v-card class="ma-5 pa-5">
        <div>
          <p class="text-h6 ma-0 primary--text">TOTAL MEMBER EXPIRED</p>
          <p class="text-body-1 mb-2">Jumlah member expired</p>
          <div v-if="loading" class="text-medium ma-0">
           Loading...
          </div>
          <h1 v-else class="text-h3 ma-0 font-weight-bold">
            {{ addCommas(homeTotal.expired_members) }}
          </h1>
        </div>
      </v-card>
    </v-col>
    <v-col cols="12" lg="4" xl="3">
      <v-card class="ma-5 pa-5">
        <div>
          <p class="text-h6 ma-0 primary--text">TOTAL DATA KENDARAAN</p>
          <p class="text-body-1 mb-2">Jumlah data kendaraan</p>
          <div v-if="loading" class="text-medium ma-0">
           Loading...
          </div>
          <h1 v-else class="text-h3 ma-0 font-weight-bold">
            {{ addCommas(homeTotal.kendaraan) }}
          </h1>
        </div>
      </v-card>
    </v-col>
    <!-- <v-col cols="12" lg="4" xl="3">
      <v-card class="ma-5 pa-5">
        <div>
          <p class="text-h6 ma-0 primary--text">KONFIRMASI PEMBAYARAN</p>
          <p class="text-body-1 mb-2">Total konfirmasi pembayaran</p>
          <h1 class="text-h3 ma-0 font-weight-bold">
            12
          </h1>
        </div>
      </v-card>
    </v-col> -->
  </v-row>
  <v-row>
    <div v-if="homeTotal.leasing_chart === null">
    </div>
    <v-col 
    v-else
    cols="12" lg="6">
      <v-card class="mx-3 ml-3 pa-10">
        <div class="text-medium font-weight-bold">
          Data Kendaraan Per Data
        </div>
        <v-data-table
          :headers="headers"
          :items="leasing_chart"
          hide-default-footer
          disable-pagination
        >
        </v-data-table>
        <!-- <LeasingChart/> -->
      </v-card>
    </v-col>
  </v-row>
</div>
</template>

<script>
import LeasingChart from '../components/Chart/LeasingChart.vue';
export default {
    name: "IndexPage",
    middleware: "auth",
    computed:{
      headers() {
        return [
          { text: "Nama Data", value: "leasing_name" },
          { text: "Jumlah Kendaraan", value: "count" },
        ];
      },
    },
    data() {
        return {
            loading: false,
            leasing_chart: [],
            homeTotal: {
                leasing: 0,
                expired_members: 0,
                premium_members: 0,
                trial_members: 0,
            },
        };
    },
    mounted() {
        this.fetchData();
        this.fetchKendaraanPerCabang();
    },
    methods: {
        fetchData() {
            this.loading = true;
            this.$axios
                .get("home")
                .then((response) => {
                this.homeTotal = response.data.data;
                this.loading = false;
            })
                .catch((error) => {
                this.loading = false;
            })
                .finally(() => {
                this.loading = false;
            });
        },
        fetchKendaraanPerCabang() {
            this.loading = true;
            this.$axios
                .get("kendaraan-per-cabang")
                .then((response) => {
                  this.leasing_chart = response.data.data;
                  this.loading = false;
            })
                .catch((error) => {
                this.loading = false;
            })
                .finally(() => {
                this.loading = false;
            });
        },
         addCommas(n){
          var rx=  /(\d+)(\d{3})/;
          return String(n).replace(/^\d+/, function(w){
              while(rx.test(w)){
                  w= w.replace(rx, '$1.$2');
              }
              return w;
          });
}
    },
    components: { LeasingChart }
};
</script>
