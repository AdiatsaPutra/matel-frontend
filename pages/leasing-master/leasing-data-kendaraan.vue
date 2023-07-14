<template>
  <div>
    <div v-if="!isDetail">
      <div>Data Kendaraan {{ leasingName }}</div>
      <v-row class="pt-5 mx-1">
        <v-select
          v-model="selectedCabang"
          :items="cabangFilter"
          :disabled="loading"
          solo
          dense
          item-text="nama_cabang"
          item-value="id"
          placeholder="Filter Cabang"
          @change="selectCabang(selectedCabang)"
        ></v-select>
        <div class="mx-2"></div>
        <v-text-field
          v-model="search"
          placeholder="Cari berdasarkan leasing, cabang, atau nomor polisi"
          solo
          dense
          prepend-inner-icon="mdi-magnify"
          @input="debouncedFetchLeasing"
        ></v-text-field>
      </v-row>
    </div>
    <v-card v-else>
      <v-row class="pa-5">
        <v-col cols="12">
          <div class="text-h6">Detail Leasing</div>
          <div class="mb-5"></div>
          <v-row>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.leasing"
                label="Leasing"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.cabang"
                label="Cabang"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.no_kontrak"
                label="No. Kontrak"
                readonly
                outlined
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.nama_debitur"
                label="Nama Debitur"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.nomor_polisi"
                label="Nomor Polisi"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.sisa_hutang"
                label="Sisa Hutang"
                readonly
                outlined
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.tipe"
                label="Tipe"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.tahun"
                label="Tahun"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.no_rangka"
                label="No. Rangka"
                readonly
                outlined
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.no_mesin"
                label="No. Mesin"
                readonly
                outlined
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="selectedLeasing.pic"
                label="PIC"
                readonly
                outlined
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-spacer></v-spacer>
            <v-btn color="primary" class="ma-5" dark @click="isDetail = false"
              >Kembali</v-btn
            >
          </v-row>
        </v-col>
      </v-row>
    </v-card>
    <!-- <div class="text-body-2 px-2 mb-2" v-if="!isDetail">
      Total Data: {{ total }}
    </div> -->

    <v-data-table
      v-if="!isDetail"
      :headers="headers"
      :items="numberedItems"
      :search="search"
      :loading="loading"
    >
      <template v-slot:item.sisa_hutang="{ item }">
        {{ formatCurrency(item.sisa_hutang) }}
      </template>
      <template v-slot:item.actions="{ item }">
        <v-btn color="primary" height="27px" dark @click="viewDetail(item.id)">
          Detail
        </v-btn>
        <!-- <v-btn color="red" height="27px" dark @click="deleteItem(item.id)">
          Hapus
        </v-btn> -->
      </template>
    </v-data-table>
  </div>
</template>

<script>
import { debounce } from "lodash";

export default {
  props: {
    leasingId: 0,
    leasingName: "",
    cabangName: "",
  },
  data() {
    return {
      items: [],
      cabang: [],
      cabangFilter: [
        {
          'id': '',
          'nama_cabang': 'All'
        }
      ],
      loading: false,
      pagination: {
        page: 1,
        itemsPerPage: 100,
      },
      options: {},
      totalPages: 10,
      total: 10,
      search: "",
      currentPage: 1,
      limit: -1,
      debouncedFetchLeasing: debounce(this.fetchLeasing, 300),
      isDetail: false,
      selectedLeasing: null,
      selectedCabang: null,
      selectedDownloadCabang: null,
      selectedGantikanDataCabang: null,
      selectedUploadCabang: null,
      showModal: false,
      showUploadModal: false,
      isLoading: false,
      success: false,
      isError: false,
      error: null,
      file: null,
      formData: null,
      time: null,
    };
  },
  computed: {
    headers() {
      return [
        { text: "No", value: "id" },
        { text: "Leasing", value: "leasing" },
        { text: "Cabang", value: "cabang" },
        { text: "Nama Debitur", value: "nama_debitur" },
        { text: "No Polisi", value: "nomor_polisi" },
        { text: "Sisa Hutang", value: "sisa_hutang" },
        { text: "Actions", value: "actions" },
      ];
    },
    numberedItems() {
      const startIndex = (this.currentPage - 1) * this.limit;
      return this.items.map((item, index) => ({
        ...item,
        id: index + 1,
      }));
    },
  },
  mounted() {
    this.fetchLeasing();
    this.$store.watch(
      (state) => state.myString,
      (newString) => {
        if (newString === "Cabang Added") {
          this.fetchCabang();
        } else if (newString === "Kendaraan Added") {
          this.fetchLeasing();
        }
      }
    );
    this.fetchCabang();
    this.fetchLeasingTotal();
  },
  methods: {
    fetchCabang() {
      this.loading = true;
      this.$axios
        .get("cabang", {
          params: {
            leasing_id: this.leasingId,
            search: this.search,
            page: this.options.page,
            limit: this.limit,
          },
        })
        .then((response) => {
          this.cabang = response.data.data.cabang;
          this.cabangFilter.push.apply(this.cabangFilter, this.cabang)
        })
        .catch((error) => {
          console.error(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    fetchLeasingTotal() {
      this.loading = true;
      this.$axios
        .get("home", {
          params: {
            leasing: this.leasingName,
            cabang: this.cabangName,
          }
        })
        .then((response) => {
          this.total = response.data.data.leasing;
          console.log(response.data.data)
          this.loading = false;
        })
        .catch((error) => {
          console.error(error);
          this.loading = false;
        })
        .finally(() => {
          this.loading = false;
        });
    },
    fetchLeasing() {
      this.loading = true;
      this.$axios
        .get("kendaraan", {
          params: {
            leasing: this.leasingName,
            search: this.search,
            page: this.options.page,
            limit: this.limit,
            cabang: this.cabangName,
          },
        })
        .then((response) => {
          this.items = response.data.data;
          this.totalPages = Math.ceil(response.data.data.total / this.limit);
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    deleteKendaraan() {
      this.loading = true;

      this.$axios
        .delete("delete-kendaraan", {
          params: {
            leasing_id: this.leasingId,
            leasing: this.leasingName,
            cabang: this.selectedGantikanDataCabang,
          },
        })
        .then((response) => {
          if (this.formData) {
            const cabangFiltered = this.cabang.filter(
              (item) => item.nama_cabang === this.selectedGantikanDataCabang
            );
            const cabangName = cabangFiltered[0].nama_cabang;
            this.formData.append("cabang_name", cabangName);
            this.success = false;
            this.isError = false;
            this.isLoading = true;
            this.$axios
              .post("upload-leasing", this.formData, {
                headers: {
                  "Content-Type": "multipart/form-data",
                },
              })
              .then((response) => {
                this.showGantikanData = false;
                this.getLeasing();
              })
              .catch((error) => {
                this.showGantikanData = false;
                this.isError = true;
                this.isLoading = false;
                this.error = error.message;
                this.fetchLeasing();
              });
          }
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    formatCurrency(value) {
      const formatter = new Intl.NumberFormat("id-ID", {
        style: "currency",
        currency: "IDR",
      });
      return formatter.format(value);
    },
    viewDetail(itemId) {
      this.selectedLeasing = this.items.find((item) => item.id === itemId);
      this.isDetail = true;
    },
    selectCabang(item) {
      this.selectedCabang = item;
      const cabangFiltered = this.cabangFilter.filter(
        (item) => item.id === this.selectedCabang
      );
      this.cabangName = cabangFiltered[0].nama_cabang === 'All' ? '' : cabangFiltered[0].nama_cabang;
      this.fetchLeasing();
    },
    editItem(itemId) {},
    deleteItem(itemId) {},
    showDownloadModal() {
      this.showModal = !this.showModal;
      this.selectedDownloadCabang = null;
    },
    showGantikanDataModal() {
      this.gantikanData = !this.gantikanData;
      this.selectedGantikanDataCabang = null;
    },
  },
};
</script>
