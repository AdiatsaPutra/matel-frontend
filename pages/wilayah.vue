<template>
    <v-container fluid>
      <div class="text-medium font-weight-bold mb-2">Data Provinsi</div>
      <v-row>
        <v-col>
          <v-text-field
            v-model="searchProvince"
            placeholder="Cari Provinsi"
            dense
            solo
            prepend-inner-icon="mdi-magnify"
          ></v-text-field>
        </v-col>
      </v-row>
  
      <v-data-table :headers="headers" :items="filteredProvince">
        <template v-slot:item.actions="{ item }">
          <v-btn color="primary" height="30px" dark @click="openDetailModal(item)">
            Detail
          </v-btn>
          <v-btn color="red" height="30px" dark @click="openConfirmModal(item)">
            Hapus
          </v-btn>
        </template>
      </v-data-table>
      <div class="mt-5"></div>
      <div class="text-medium font-weight-bold mb-2">Data Kabupaten</div>
      <v-row>
        <v-col>
          <v-select
            v-model="selectedProvince"
            :items="province"
            item-text="name"
            item-value="id"
            label="Provinsi"
            dense
            solo
          ></v-select>
        </v-col>
        <v-col>
          <v-text-field
            v-model="searchKabupaten"
            placeholder="Cari Kabupaten"
            dense
            solo
            prepend-inner-icon="mdi-magnify"
          ></v-text-field>
        </v-col>
      </v-row>
  
      <v-data-table :headers="headers" :items="filteredKabupaten">
        <template v-slot:item.actions="{ item }">
          <v-btn color="primary" height="30px" dark @click="openDetailModal(item)">
            Detail
          </v-btn>
          <v-btn color="red" height="30px" dark @click="openConfirmModal(item)">
            Hapus
          </v-btn>
        </template>
      </v-data-table>
      <div class="mt-5"></div>
      <div class="text-medium font-weight-bold mb-2">Data Kecamatan</div>
      <v-row>
        <v-col>
          <v-select
            v-model="selectedKabupaten"
            :items="kabupaten"
            item-text="name"
            item-value="id"
            label="Kabupaten"
            dense
            solo
          ></v-select>
        </v-col>
        <v-col>
          <v-text-field
            v-model="searchKecamatan"
            placeholder="Cari Kecamatan"
            dense
            solo
            prepend-inner-icon="mdi-magnify"
          ></v-text-field>
        </v-col>
      </v-row>
  
      <v-data-table :headers="headers" :items="filteredKecamatan">
        <template v-slot:item.actions="{ item }">
          <v-btn color="primary" height="30px" dark @click="openDetailModal(item)">
            Detail
          </v-btn>
          <v-btn color="red" height="30px" dark @click="openConfirmModal(item)">
            Hapus
          </v-btn>
        </template>
      </v-data-table>
  
      <v-dialog v-model="isDetailModalOpen" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">Detail</span>
          </v-card-title>
          <v-card-text>
            <v-text-field
              v-model="selectedData.name"
              label="Name"
              readonly
            ></v-text-field>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" dark @click="closeDetailModal">Tutup</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
  
      <v-dialog v-model="isConfirmModalOpen" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">Konfirmasi Hapus</span>
          </v-card-title>
          <v-card-text>
            Apakah Anda yakin ingin menghapus data ini?
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" dark @click="deleteData(selectedData)"
              >Ya</v-btn
            >
            <v-btn color="red" dark @click="closeConfirmModal">Tidak</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </template>
  
  <script>
  export default {
    data() {
      return {
        province: [],
        kabupaten: [],
        kecamatan: [],
        searchProvince: "",
        searchKabupaten: "",
        searchKecamatan: "",
        selectedProvince: 0,
        selectedKabupaten: 0,
        isDetailModalOpen: false,
        isConfirmModalOpen: false,
        selectedData: {},
        headers: [
          { text: "ID", value: "id" },
          { text: "Name", value: "name" },
          { text: "Actions", value: "actions", sortable: false },
        ],
      };
    },
    computed: {
      filteredProvince() {
        return this.province.filter((item) =>
          item.name.toLowerCase().includes(this.searchProvince.toLowerCase())
        );
      },
      filteredKabupaten() {
        if (this.selectedProvince) {
          return this.kabupaten.filter(
            (item) =>
              item.province_id === this.selectedProvince &&
              item.name
                .toLowerCase()
                .includes(this.searchKabupaten.toLowerCase())
          );
        }
        return [];
      },
      filteredKecamatan() {
        if (this.selectedKabupaten) {
          return this.kecamatan.filter(
            (item) =>
              item.kabupaten_id === this.selectedKabupaten &&
              item.name
                .toLowerCase()
                .includes(this.searchKecamatan.toLowerCase())
          );
        }
        return [];
      },
    },
    methods: {
      fetchProvince() {
        this.$axios
          .get("province")
          .then((response) => {
            if (response.data.data === null) {
              this.province = [];
            } else {
              this.province = response.data.data;
            }
          })
          .catch((error) => {
            console.error(error);
          });
      },
      fetchKabupaten() {
        if (this.selectedProvince) {
          this.$axios
            .get(`kabupaten/${this.selectedProvince}`)
            .then((response) => {
              if (response.data.data === null) {
                this.kabupaten = [];
              } else {
                this.kabupaten = response.data.data;
              }
            })
            .catch((error) => {
              console.error(error);
            });
        }
      },
      fetchKecamatan() {
        if (this.selectedKabupaten) {
          this.$axios
            .get(`kecamatan/${this.selectedKabupaten}`)
            .then((response) => {
              if (response.data.data === null) {
                this.kecamatan = [];
              } else {
                this.kecamatan = response.data.data;
              }
            })
            .catch((error) => {
              console.error(error);
            });
        }
      },
      openDetailModal(data) {
        this.selectedData = data;
        this.isDetailModalOpen = true;
      },
      closeDetailModal() {
        this.isDetailModalOpen = false;
      },
      openConfirmModal(data) {
        this.selectedData = data;
        this.isConfirmModalOpen = true;
      },
      closeConfirmModal() {
        this.isConfirmModalOpen = false;
      },
      deleteData(data) {
        this.$axios
          .delete(`delete-data/${data.id}`)
          .then((response) => {
            this.fetchData();
          })
          .catch((error) => {
            console.error(error);
          });
        this.closeConfirmModal();
      },
    },
    watch: {
      selectedProvince() {
        this.fetchKabupaten();
        this.selectedKabupaten = null;
      },
      selectedKabupaten() {
        this.fetchKecamatan();
      },
    },
    mounted() {
      this.fetchProvince();
      this.fetchKabupaten();
      this.fetchKecamatan();
    },
  };
  </script>