<template>
  <div>
    <div v-if="!isDetail">
      <div>Data Kendaraan</div>
      <v-row class="pt-5 mx-1">
        
        <v-btn height="40px" color="primary" @click="uploadModal(false)"
          >Upload Data Kendaraan</v-btn
        >
        <div class="mx-2"></div>
        <v-btn height="40px" color="orange" dark @click="uploadModal(true)"
        >Ganti Semua Data</v-btn
        >
        <div class="mx-2"></div>
        <v-btn height="40px" color="purple" dark @click="downloadTemplate"
        >Download Template</v-btn
        >
        <div class="mx-2"></div>
        <v-text-field
          v-model="search"
          placeholder="Cari berdasarkan leasing, cabang atau nomor polisi"
          solo
          dense
          prepend-inner-icon="mdi-magnify"
          @input="fetchLeasing"
        ></v-text-field>
      </v-row>
    </div>
    <v-card v-else class="mt-5">
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
    <div class="text-medium mb-2" v-if="!isDetail">
      Total Data: {{ loading ? 'Loading...' : total }}
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
    <template v-slot:item.id="{ item, index }">
      <td>{{ index + 1 }}</td> 
    </template>
      <template v-slot:item.sisa_hutang="{ item }">
        {{ formatCurrency(item.sisa_hutang) }}
      </template>
      <template v-slot:item.actions="{ item }">
        <v-btn color="primary" height="27px" dark @click="viewDetail(item.id)">
          <div class="text-caption">
            Detail
          </div>
        </v-btn>
        <!-- <v-btn color="red" height="27px" dark @click="deleteItem(item.id)">
          <div class="text-caption">
            Hapus
          </div>
        </v-btn> -->
      </template>
      <template v-slot:footer>
          <v-pagination
          class="py-5"
          v-model="currentPage"
          :length="Math.ceil(total / perPage)"
          prev-icon="mdi-chevron-left"
          next-icon="mdi-chevron-right"
          :total-visible="12"
          :disabled="loading"
          @input="handlePageChange"
        ></v-pagination>
        </template>
    </v-data-table>

    <v-dialog v-model="showModal" max-width="500">
      <v-card class="pa-5">
        <div class="text-h6">Download Template</div>

        <div class="mb-5"></div>
        <v-row>
          <v-spacer></v-spacer>
          <v-btn color="red" dark @click="showDownloadModal">Batal</v-btn>
          <div class="mx-2"></div>
          <v-btn color="primary" @click="downloadTemplate">Download</v-btn>
        </v-row>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showUploadModal" max-width="500">
      <v-card class="pa-5">
        <div class="text-h6 purple--text text--darken-4">UPLOAD DATA</div>
        <div v-if="isGantikanData" class="text-medium red--text">
          Gantikan data akan mengganti seluruh data kendaraan saat ini
        </div>

        <v-alert v-show="isError === true" type="error">
          <div>
            <div class="text-subtitle-1 text--black">
              {{ error }}
            </div>
          </div>
        </v-alert>
        <div class="py-1"></div>

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
            @click="uploadModal"
          >
            Batal
          </v-btn>
          <div class="mx-2"></div>
          <v-btn
          v-if="isGantikanData"
            color="primary white--text"
            height="32"
            :disabled="isLoading || success || file === null"
            :loading="isLoading"
            @click="updateAllKendaraan"
          >
            Upload
          </v-btn>
          <v-btn
          v-else
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
  </div>
</template>

<script>
import { debounce } from "lodash";

export default {
  data() {
    return {
      items: [],
      loading: false,
      options: {},
      total: 0,
      search: "",
      isGantikanData: false,
      currentPage: 1,
      perPage: 20,
      limit: -1,
      debouncedFetchLeasing: debounce(this.fetchLeasing, 300),
      isDetail: false,
      selectedLeasing: null,
      selectedCabang: null,
      selectedDownloadCabang: null,
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
   
  },
  mounted() {
    this.fetchLeasingTotal();
    this.fetchLeasing();
  },
  methods: {
    handlePageChange(page) {
    this.currentPage = page;

    this.fetchLeasing();
  },
    fetchLeasingTotal() {
      this.loading = true;
      this.$axios
        .get("home")
        .then((response) => {
          this.total = response.data.data.kendaraan;
          this.loading = false;
        })
        .catch((error) => {
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
            search: this.search,
            page: this.currentPage,
            limit: this.perPage,
      },
        })
        .then((response) => {
          this.items = [];
          this.items = response.data.data;
        })
        .catch((error) => {
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
    },
    uploadFile() {
      if (this.formData) {
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
            this.formData = null;
            this.success = true;
            this.isLoading = false;
            this.showUploadModal = false;
            this.selectedUploadCabang = null;
            this.file = null;
            this.fetchLeasingTotal()
            this.fetchLeasing()
          })
          .catch((error) => {
            this.showUploadModal = false;
            this.isError = true;
            this.isLoading = false;
            this.error = error.message;
          });
      }
    },
    downloadTemplate() {
      const endpoint = "/download-template";
      const url = this.$axios.defaults.baseURL + endpoint;

      this.$axios
        .get("download-template")
        .then((response) => {
          const downloadLink = document.createElement("a");
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const filename = "leasing-template.csv";
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
    editItem(itemId) {},
    updateAllKendaraan() {
      this.loading = true;
      this.$axios
        .delete("delete-all-kendaraan")
        .then((response) => {
          this.uploadFile()
        })
        .catch((error) => {
        })
        .finally(() => {
          this.loading = false;
        });
    },
    uploadModal(data) {
      this.isGantikanData = data
      this.showUploadModal = !this.showUploadModal;
      this.file = null;
      this.success = false;
    },
    showDownloadModal() {
      this.showModal = !this.showModal;
      this.selectedDownloadCabang = null;
    },
  },
};
</script>
