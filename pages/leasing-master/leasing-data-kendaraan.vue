<template>
  <div>
    <div v-if="!isDetail">
      <div>Data Kendaraan {{ leasingName }}</div>
      <v-row class="pt-5 mx-1">
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
                v-model="selectedLeasing.cabangData"
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
    <div class="text-body-2 px-2 mb-2" v-if="!isDetail">
      Total Data: {{ kendaraanTotal }}
    </div>

    <v-data-table
      v-if="!isDetail"
      :headers="headers"
      :items="items"
      :search="search"
      :loading="loading"
      hide-default-footer
      disable-pagination
    >
      <template v-slot:item.CreatedAt="{ item }">
        {{ new Intl.DateTimeFormat("id-ID", { day: "numeric", month: "long", year: "numeric" }).format(items.CreatedAt) }}
      </template>
      <template v-slot:item.actions="{ item }">
        <v-btn color="primary" height="27px" dark @click="viewDetail(item.id)">
          Detail
        </v-btn>
        <v-btn color="red" height="27px" dark @click="showDeleteKendaraanDialog(item.id)">
          Hapus
        </v-btn>
      </template>
      <template v-slot:footer>
          <v-pagination
          class="py-5"
          v-model="currentPage"
          :length="Math.ceil(kendaraanTotal / perPage)"
          prev-icon="mdi-chevron-left"
          next-icon="mdi-chevron-right"
          :total-visible="12"
          :disabled="loading"
          @input="handlePageChange"
        ></v-pagination>
        </template>
    </v-data-table>

    <v-dialog v-model="showDeleteDialog" max-width="500">
    <v-card>
      <div class="pt-5"></div>
      <div class="text-medium px-5">
        Anda yakin akan menghapus data ini?
      </div>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" height="27px" dark @click="showDeleteDialog = false">
          Batal
        </v-btn>
        <v-btn color="red" height="27px" dark @click="deleteKendaraan">
          Hapus
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
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
      perPage: 20,
      limit: -1,
      kendaranLimit: 20,
      debouncedFetchLeasing: debounce(this.fetchLeasing, 300),
      isDetail: false,
      selectedLeasing: null,
      kendaraanTotal: 0,
      selectedCabang: null,
      selectedDownloadCabang: null,
      selectedUploadCabang: null,
      showModal: false,
      showDeleteDialog: false,
      itemToDelete: null,
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
        { text: "Nama Leasing", value: "leasing" },
        { text: "Cabang", value: "cabangData" },
        { text: "Nama Debitur", value: "nama_debitur" },
        { text: "No Polisi", value: "nomor_polisi" },
        { text: "Tanggal Upload", value: "CreatedAt" },
        { text: "Actions", value: "actions" },
      ];
    },
    numberedItems() {
      return this.items.map((item, index) => ({
        ...item,
        no: index + 1,
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
    handlePageChange(page) {
    this.currentPage = page;
    this.fetchLeasing();
  },
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
            page: this.currentPage,
            limit: this.perPage,
            cabang: this.cabangName,
          },
        })
        .then((response) => {
          this.items = response.data.data.kendaraan;
          console.log(response.data.data)
          this.kendaraanTotal = response.data.data.total
          this.totalPages = Math.ceil(response.data.data.total / this.limit);
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
      console.log(this.items[0])
      console.log(itemId)
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
    showDeleteKendaraanDialog(itemId) {
      this.$store.dispatch("updateString", "");
      this.itemToDelete = itemId;
      this.showDeleteDialog = true;
    },
    deleteKendaraan() {
      this.loading = true;
      this.$axios
      .delete(`delete-kendaraan/${this.itemToDelete}`)
      .then((response) => {
          this.fetchLeasing();
          this.fetchCabang();
          this.fetchLeasingTotal();
          this.showDeleteDialog = false;
          this.$store.dispatch("updateString", "Kendaraan Added");
          this.$store.dispatch("updateString", "Cabang Added");
        })
        .catch((error) => {
        })
        .finally(() => {
          this.loading = false;
        });
    },
    showDownloadModal() {
      this.showModal = !this.showModal;
      this.selectedDownloadCabang = null;
    },
    
  },
};
</script>
