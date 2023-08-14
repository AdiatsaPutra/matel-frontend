<template>
  <div>
    <!-- Table view -->
    <div v-if="!showDetail">
      <v-row class="mt-5 mx-1">
        <v-btn class="mb-4" color="primary" @click="openCreateDialog(true)">
          Tambah Leasing
        </v-btn>
        <v-spacer></v-spacer>
        <v-text-field v-model="search" label="Cari" solo dense></v-text-field>
      </v-row>
      <v-data-table
        :headers="headers"
        :items="items"
        :search="search"
        :footer-props="{
          'items-per-page-options': [10, 20, 50, 100, -1]
        }"
        :items-per-page="20"
        :loading="loading"
      >
        <template v-slot:item.actions="{ item }">
          <v-btn
            color="primary"
            height="27px"
            min-width="60px"
            @click="showLeasingDetail(item)"
          >
            <div class="text-caption">Detail</div>
          </v-btn>
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

    <!-- Detail view -->
    <div v-else>
      <v-card class="pa-5" >
        <v-btn
          class="py-0"
          outlined
          @click="showDetail = false"
          color="primary"
        >
          <v-icon>mdi-arrow-left-thin</v-icon>
          <div>Kembali</div>
        </v-btn>
        <div class="mb-2"></div>
        <div class="text-h6">Detail Leasing</div>
        <div class="mb-2"></div>
        <v-row>
          <v-col cols="12" sm="4">
            <div class="text-body">Nama Leasing</div>
            <div class="text-body font-weight-bold">{{ selectedLeasing.nama_leasing }}</div>
          </v-col>
          <v-col cols="12" sm="4">
            <div class="text-body">Nama PIC</div>
            <div class="text-body font-weight-bold">{{ selectedLeasing.nama_pic }}</div>
          </v-col>
          <v-col cols="12" sm="4">
            <div class="text-body">No HP PIC</div>
            <div class="text-body font-weight-bold">{{ selectedLeasing.no_hp_pic }}</div>
          </v-col>
        </v-row>
      </v-card>
      <div class="mb-10"></div>
      <LeasingMasterDetail 
      :leasingId="selectedLeasing.id" 
      :leasingName="selectedLeasing.nama_leasing"
      />
      <div class="mb-10"></div>
      <DataKendaraan
        :leasingId="selectedLeasing.id"
        :leasingName="selectedLeasing.nama_leasing"
      />
    </div>

    <v-dialog v-model="createDialog" max-width="1000px" max-height="1000px">
      <v-card>
        <v-card-title>Tambah Leasing</v-card-title>
        <v-card-text>
          <v-form ref="createForm">
            <v-text-field
              v-model="newLeasing.nama_leasing"
              :rules="nameRules"
              outlined
              label="Nama Leasing"
            ></v-text-field>
            <v-text-field
              v-model="newLeasing.nama_pic"
              :rules="nameRules"
              outlined
              label="Nama PIC"
            ></v-text-field>
            <v-text-field
              v-model="newLeasing.no_hp_pic"
              :rules="nameRules"
              outlined
              label="No HP PIC"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" min-width="160px" @click="createLeasing">
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
        <v-card-title>Edit Leasing</v-card-title>
        <v-card-text>
          <v-form ref="editForm">
            <v-text-field
              v-model="editLeasing.nama_leasing"
              :rules="nameRules"
              outlined
              label="Nama Leasing"
            ></v-text-field>
            <v-text-field
              v-model="editLeasing.nama_pic"
              :rules="nameRules"
              outlined
              label="Nama PIC"
            ></v-text-field>
            <v-text-field
              v-model="editLeasing.no_hp_pic"
              :rules="nameRules"
              outlined
              label="No HP PIC"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" min-width="160px" @click="updateLeasing"
            >Edit</v-btn
          >
          <v-btn color="secondary" min-width="160px" @click="cancelEdit"
            >Batal</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="deleteDialog" max-width="400px">
      <v-card>
        <v-card-title>Konfirmasi Hapus</v-card-title>
        <v-card-text>Apakah anda yakin akan menghapus data Leasing, Semua data Leasing terkait akan dihapus?</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="secondary" @click="cancelDelete">Batal</v-btn>
          <v-btn color="red" dark @click="deleteItem">Hapus</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import LeasingMasterDetail from "@/pages/leasing-master/leasing-master-detail.vue";
import DataKendaraan from "@/pages/leasing-master/leasing-data-kendaraan.vue";

export default {
  components: {
    LeasingMasterDetail,
    DataKendaraan,
  },
  data() {
    return {
      items: [],
      headers: [
        { text: "Nama Leasing", value: "nama_leasing" },
        { text: "Nama PIC", value: "nama_pic" },
        { text: "Nomor HP PIC", value: "no_hp_pic" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      search: "",
      total: "",
      limit: 10,
      options: {},
      loading: false,
      createDialog: false,
      editDialog: false,
      deleteDialog: false,
      selectedItem: null,
      showDetail: false,
      selectedLeasing: null,
      newLeasing: {
        nama_leasing: "",
        nama_pic: "",
        no_hp_pic: "",
      },
      editLeasing: {
        id: "",
        nama_leasing: "",
        nama_pic: "",
        no_hp_pic: "",
      },
      nameRules: [(v) => !!v || "Wajib diisi"],
    };
  },
  methods: {
    fetchData() {
      this.loading = true;
      this.$axios
        .get("leasing-master", {
          params: {
            search: this.search,
            page: this.options.page,
            limit: -1,
          },
        })
        .then((response) => {
          this.items = response.data.data.leasing;
          this.total = response.data.data.total;
        })
        .catch((error) => {
          console.error(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    fetchDataKendaraan() {
      this.loading = true;
      this.$axios
        .get("kendaraan", {
          params: {
            search: this.search,
            page: this.options.page,
            limit: this.limit,
          },
        })
        .then((response) => {
          this.items = response.data.data.leasing;
          this.total = response.data.data.total;
        })
        .catch((error) => {
          console.error(error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    createLeasing() {
      if (this.$refs.createForm.validate()) {
        this.$axios
          .post("leasing-master", this.newLeasing)
          .then((response) => {
            this.$refs.createForm.reset();
            this.fetchData();
            this.newLeasing = {
              nama_leasing: "",
              nama_pic: "",
              no_hp_pic: "",
            };
            this.createDialog = false;
          })
          .catch((error) => {
            console.error(error);
          });
      }
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
              nama_leasing: "",
              nama_pic: "",
              no_hp_pic: "",
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
};
</script>
