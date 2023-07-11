<template>
  <div>
    <div v-if="!isDetail">
      <div>Data Kendaraan {{ leasingName }}</div>
      <v-row class="pt-5 mx-1">
        <v-btn
          v-if="cabang.length > 0"
          height="40px"
          color="primary"
          @click="showUpload"
          >Upload Data Kendaraan</v-btn
        >
        <div v-if="cabang.length > 0" class="mx-2"></div>
        <v-btn
        v-if="cabang.length > 0"
        height="40px"
        color="red"
        dark
        @click="showGantikanData = true"
        >Gantikan Data</v-btn
        >
        <div v-if="cabang.length > 0" class="mx-2"></div>
        <v-btn
          v-if="cabang.length > 0"
          height="40px"
          color="purple"
          dark
          @click="downloadTemplate"
          >Download Template</v-btn
        >
        <div v-if="cabang.length > 0" class="mx-2"></div>
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

    <div>
      <v-alert
        v-if="cabang.length === 0"
        v-model="alert"
        tonal
        close-label="Close Alert"
        color="warning"
        dark
        title="Closable Alert"
      >
        Tambah cabang sebelum upload
      </v-alert>
    </div>

    <v-data-table
      v-if="!isDetail"
      :headers="headers"
      :items="items"
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

    <v-dialog v-model="showUploadModal" max-width="500">
      <v-card class="pa-5">
        <div class="text-h6 purple--text text--darken-4">UPLOAD DATA</div>
        <div class="py-1"></div>

        <v-alert v-show="isError === true" type="error">
          <div>
            <div class="text-subtitle-1 text--black">
              {{ error }}
            </div>
          </div>
        </v-alert>
        <div class="py-1"></div>

        <v-select
          v-model="selectedUploadCabang"
          :items="cabang"
          item-text="nama_cabang"
          item-value="id"
          solo
          dense
          placeholder="Pilih Cabang"
        ></v-select>

        <v-file-input
          v-model="file"
          multiple
          dense
          placeholder="Pilih File"
          solo
          prepend-icon
          @change="handleFileChange"
        ></v-file-input>
        <v-row>
          <v-spacer></v-spacer>
          <v-btn
            color="red white--text"
            height="32"
            @click="showUploadModal = false"
          >
            Batal
          </v-btn>
          <div class="mx-2"></div>
          <v-btn
            color="primary white--text"
            height="32"
            :disabled="isLoading || success || file === null"
            :loading="isLoading"
            @click="uploadFile"
          >
            Upload
          </v-btn>
        </v-row>
        <div class="py-2"></div>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showModal" max-width="500">
      <v-card class="pa-5">
        <div class="text-h6">Download Template</div>

        <v-row>
          <v-spacer></v-spacer>
          <v-btn color="red" dark @click="showDownloadModal">Batal</v-btn>
          <div class="mx-2"></div>
          <v-btn color="primary" @click="downloadTemplate">Download</v-btn>
        </v-row>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showGantikanData" max-width="500">
      <v-card class="pa-5">
        <div class="text-h6">Gantikan Data</div>

        <div class="mb-5"></div>
        <v-file-input
          v-model="file"
          multiple
          dense
          placeholder="Pilih File"
          solo
          prepend-icon
          @change="handleFileChange"
        ></v-file-input>
        <div class="mb-2"></div>
        <v-select
          v-model="selectedGantikanDataCabang"
          :items="cabang"
          item-text="nama_cabang"
          item-value="nama_cabang"
          solo
          dense
          placeholder="Pilih Cabang"
        ></v-select>
        <div class="mb-5"></div>
        <v-row>
          <v-spacer></v-spacer>
          <v-btn color="red" dark @click="showGantikanData = false"
            >Batal</v-btn
          >
          <div class="mx-2"></div>
          <v-btn color="primary" @click="deleteKendaraan">Gantikan Data</v-btn>
        </v-row>
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
      limit: 100,
      debouncedFetchLeasing: debounce(this.fetchLeasing, 300),
      isDetail: false,
      selectedLeasing: null,
      selectedCabang: null,
      selectedDownloadCabang: null,
      selectedGantikanDataCabang: null,
      selectedUploadCabang: null,
      showModal: false,
      showGantikanData: false,
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
        id: startIndex + index + 1,
      }));
    },
  },

  mounted() {
    this.fetchLeasing();
    this.fetchCabang();
    this.fetchLeasingTotal();
    this.$store.watch(
      (state) => state.myString,
      (newString) => {
        if (newString === "Cabang Added") {
          this.fetchCabang();
        }
      }
    );
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
        .get("home")
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
    handleFileChange() {
      this.formData = new FormData();

      if (this.file) {
        this.formData.append("file", this.file[0]);
      }
      this.formData.append("leasing_name", this.leasingName);
    },
    uploadFile() {
      if (this.formData) {
        const cabangFiltered = this.cabang.filter(
          (item) => item.id === this.selectedUploadCabang
        );
        const cabangName = cabangFiltered[0].nama_cabang;
        this.formData.append("cabang_name", cabangName);
        this.success = false;
        this.isError = false;
        this.isLoading = true;
        this.$axios
          .post("upload-leasing-per-cabang", this.formData, {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          })
          .then((response) => {
            this.formData = null;
            this.success = true;
            this.isLoading = false;
            this.time = response.data.data;
            this.showUploadModal = false;
            this.selectedUploadCabang = null;
            this.file = null;
            this.fetchLeasing();
          })
          .catch((error) => {
            this.showUploadModal = false;
            this.isError = true;
            this.isLoading = false;
            this.error = error.message;
            console.log("Error");
          });
      }
    },
    downloadTemplate() {
      const endpoint = "/download-template-cabang";
      const url = this.$axios.defaults.baseURL + endpoint;

      this.$axios
        .get("download-template-cabang")
        .then((response) => {
          const downloadLink = document.createElement("a");
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const filename = "leasing-template-cabang.csv";
          downloadLink.href = url;
          downloadLink.setAttribute("download", filename);
          document.body.appendChild(downloadLink);
          downloadLink.click();
          document.body.removeChild(downloadLink);
          window.URL.revokeObjectURL(url);
          this.showModal = false;
        })
        .catch((error) => {
          console.error("Failed to download file:", error);
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
                console.log("Error");
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
    showUpload() {
      this.showUploadModal = !this.showUploadModal;
      this.success = false
    },
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
