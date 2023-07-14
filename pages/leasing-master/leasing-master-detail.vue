<template>
  <div>
    <v-row class="pb-10 mx-1">
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
    </v-row>
  <v-card >
    <div class="mx-5 pt-5">
      <div class="pt-6"></div>
      <v-row class="mx-1">
        <v-btn class="mb-4" color="primary" @click="openCreateDialog(true)">
          Tambah Nama Data
        </v-btn>
        <v-spacer></v-spacer>
        <v-row>
          <v-col xs="10" lg="9">
            <v-text-field
              v-model="search"
              label="Cari"
              solo
              dense
            ></v-text-field>
          </v-col>
          <v-col xs="2" lg="2">
            <v-btn
              class="mb-4"
              height="40px"
              color="primary"
              @click="fetchData"
            >
              Cari
            </v-btn>
          </v-col>
        </v-row>
      </v-row>
      <v-data-table
        :headers="headers"
        :items="cabang"
        :search="search"
        :options.sync="options"
        :loading="loading"
      >
        <template v-slot:item.actions="{ item }">
          <v-btn
            color="secondary"
            height="27px"
            min-width="60px"
            @click="editItem(item)"
          >
            <div class="text-caption">Edit</div>
          </v-btn>
          <v-btn
            color="red"
            height="27px"
            min-width="60px"
            dark
            @click="confirmDelete(item)"
          >
            <div class="text-caption">Hapus</div>
          </v-btn>
        </template>
      </v-data-table>
    </div>

    <v-dialog v-model="createDialog" max-width="1000px" max-height="1000px">
      <v-card>
        <v-card-title>Tambah Cabang</v-card-title>
        <v-card-text>
          <v-form ref="createForm">
            <v-text-field
              v-model="newLeasing.nama_cabang"
              :rules="nameRules"
              outlined
              label="Nama Cabang"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" min-width="160px" @click="createCabang">
            Tambah
          </v-btn>
          <v-btn
            color="secondary"
            min-width="160px"
            @click="openCreateDialog(false)"
          >
            Batal
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="editDialog" max-width="1000px" max-height="1000px">
  <v-card>
    <v-card-title>Edit Cabang</v-card-title>
    <v-card-text>
      <v-form ref="editForm">
        <v-text-field
          v-model="editLeasing.nama_cabang"
          :rules="nameRules"
          outlined
          label="Nama Cabang"
        ></v-text-field>
      </v-form>
    </v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn color="primary" min-width="160px" @click="updateCabang">
        Edit
      </v-btn>
      <v-btn color="secondary" min-width="160px" @click="cancelEdit">
        Batal
      </v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>


<v-dialog v-model="deleteDialog" max-width="400px">
  <v-card>
    <v-card-title>Konfirmasi Hapus</v-card-title>
    <v-card-text> Anda yakin akan menghapus data ini? </v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn color="secondary" @click="cancelDelete">Batal</v-btn>
      <v-btn color="red" dark @click="deleteCabang">Hapus</v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>

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
            :disabled="loading || success || file === null"
            :loading="loading"
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
  </v-card>
  </div>
</template>

<script>
export default {
  props: {
    leasingId: 0,
    leasingName: "",
  },
  data() {
    return {
      cabang: [],
      headers: [
        { text: "Nama Cabang", value: "nama_cabang" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      search: "",
      total: 0,
      limit: 5,
      options: {},
      loading: false,
      createDialog: false,
      showUploadModal: false,
      editDialog: false,
      deleteDialog: false,
      showGantikanData: false,
      selectedItem: null,
      showDetail: false,
      selectedLeasing: null,
      cabangFilter: [
        {
          'id': '',
          'nama_cabang': 'All'
        }
      ],
      newLeasing: {
        leasing_id: this.leasingId,
        nama_cabang: "",
      },
      editLeasing: {
        id: "",
        nama_cabang: "",
        nama_pic: "",
        no_hp_pic: "",
      },
      nameRules: [(v) => !!v || "Wajib diisi"],
    };
  },
  computed: {
    inputValue: {
      get() {
        return this.$store.state.myString;
      },
      set(value) {
        this.$store.dispatch("updateString", value);
      },
    },
    storedString() {
      return this.$store.state.myString;
    },
  },
  methods: {
    fetchData() {
      this.loading = true;
      this.$axios
        .get("cabang", {
          params: {
            leasing_id: this.leasingId,
            search: this.search,
            page: 0,
            limit: 0,
          },
        })
        .then((response) => {
          this.cabang = response.data.data.cabang;
          this.total = response.data.data.total;
        })
        .catch((error) => {
          console.error(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    createCabang() {
      if (this.$refs.createForm.validate()) {
        this.$axios
        .post("cabang", this.newLeasing)
        .then((response) => {
          this.$store.dispatch("updateString", "");
          this.$refs.createForm.reset();
          this.fetchData();
            this.newLeasing = {
              leasing_id: this.leasingId,
              nama_cabang: "",
            };
            this.$store.dispatch("updateString", "Cabang Added");
            this.createDialog = false;
          })
          .catch((error) => {
            console.error(error);
          });
      }
    },
    updateCabang() {
    if (this.$refs.editForm.validate()) {
      const { id, ...data } = this.editLeasing;
      this.$axios
        .put(`cabang/${id}`, data)
        .then((response) => {
          this.$refs.editForm.reset();
          this.fetchData();
          this.editLeasing = {
            id: "",
            nama_cabang: "",
          };
          this.editDialog = false;
        })
        .catch((error) => {
          console.error(error);
        });
    }
  },
  deleteCabang() {
    const { id } = this.editLeasing;
    this.$axios
      .delete(`cabang/${id}`)
      .then((response) => {
        this.fetchData();
        this.deleteDialog = false;
      })
      .catch((error) => {
        console.error(error);
      });
    },
    handleFileChange() {
      this.formData = new FormData();

      if (this.file) {
        this.formData.append("file", this.file[0]);
      }
      this.formData.append("leasing_name", this.leasingName);
    },
    editItem(item) {
      this.editLeasing = { ...item };
      this.editDialog = true;
    },
    updateLeasing() {
      if (this.$refs.editForm.validate()) {
        const { id, ...data } = this.editLeasing;
        this.$axios
          .put(`leasing-master/${id}`, data)
          .then((response) => {
            this.$refs.editForm.reset();
            this.fetchData();
            this.editLeasing = {
              id: "",
              nama_cabang: "",
            };
            this.editDialog = false;
          })
          .catch((error) => {
            console.error(error);
          });
      }
    },
    cancelEdit() {
      this.editDialog = false;
    },
    uploadFile() {
      this.$store.dispatch("updateString", "");
      this.success = false;
      this.loading = true;
      if (this.formData) {
        const cabangFiltered = this.cabang.filter(
          (item) => item.id === this.selectedUploadCabang
          );
          const cabangName = cabangFiltered[0].nama_cabang;
          this.formData.append("cabang_name", cabangName);
          this.$axios
          .post("upload-leasing-per-cabang", this.formData, {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          })
          .then((response) => {
            this.formData = null;
            this.success = true;
            this.loading = false;
            this.time = response.data.data;
            this.showUploadModal = false;
            this.selectedUploadCabang = null;
            this.file = null;
            this.$store.dispatch("updateString", "Kendaraan Added");
          })
          .catch((error) => {
            this.showUploadModal = false;
            this.loading = false;
            this.error = error.message;
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
    cancelDelete() {
      this.deleteDialog = false;
    },
    confirmDelete(item) {
      this.editLeasing = { ...item };
      this.deleteDialog = true;
    },
    deleteItem() {
      const { id } = this.editLeasing;
      this.$axios
        .delete(`leasing-master/${id}`)
        .then((response) => {
          this.fetchData();
          this.deleteDialog = false;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    showUpload() {
      this.showUploadModal = !this.showUploadModal;
      this.success = false
    },
    openCreateDialog(createDialog) {
      this.createDialog = createDialog;
    },
    showLeasingDetail(item) {
      this.selectedLeasing = { ...item };
      this.showDetail = true;
    },
    
  },
  mounted() {
    this.fetchData();
  },
  watch: {
    search(newValue) {
      if (newValue === "Cabng Added") {
        this.fetchData();
      }
    },
  },
};
</script>
