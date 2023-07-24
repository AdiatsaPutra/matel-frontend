<template>
  <div>
    <v-text-field
      v-model="search"
      placeholder="Cari Bank"
      solo
      dense
      class="px-3 pt-5"
      prepend-inner-icon="mdi-magnify"
      @input="fetchBanks"
    ></v-text-field>
    <v-container fluid class="py-0">
      <v-card>
        <v-card-title>
          <v-btn color="primary" @click="openDialog(null, true, false)">Tambah Data</v-btn>
        </v-card-title>
        <v-data-table
          :headers="headers"
          :items="banks"
          :search="search"
          item-key="id"
          class="elevation-1"
        >
          <template v-slot:item.image="{ item }">
            <v-img :src="imageSrc(item.image)" alt="Base64 Image" :width="100" ></v-img>
          </template>
          <template v-slot:item.actions="{ item }">
            <v-btn color="primary" height="27px" dark @click="openDialog(item, false, false)">
              <div class="text-caption">
                Detail
              </div>
            </v-btn>
            <v-btn color="yellow" height="27px" @click="openDialog(item, false, true)">
              <div class="text-caption">
                Edit
              </div>
            </v-btn>
            <v-btn color="red" height="27px" dark @click="deleteBank(item)">
              <div class="text-caption">
                Hapus
              </div>
            </v-btn>
          </template>
        </v-data-table>
      </v-card>
    </v-container>
    <v-dialog v-model="dialog" max-width="500px">
    <v-card>
      <v-card-title>
        {{ editMode ? 'Edit Bank' : isCreate ? 'Tambah Bank' : 'Detail Bank' }}
      </v-card-title>
      <v-card-text v-if="editMode || isCreate">
        <v-select
        v-model="selectedBankData.bank_id"
        :items="bankData"
        item-text="bank" 
        item-value="id" 
        outlined
          label="Bank"
        ></v-select>
        <v-text-field v-model="selectedBankData.no_rekening" outlined label="No Rekening"></v-text-field>
      </v-card-text>
      <v-card-text v-else>
        <v-text-field v-model="selectedBankData.bank" readonly outlined label="Nama Bank"></v-text-field>
        <v-text-field v-model="selectedBankData.no_rekening" readonly outlined label="No Rekening"></v-text-field>
      </v-card-text>
      <v-card-actions v-if="editMode || isCreate">
        <v-spacer></v-spacer>
        <v-btn color="red" dark @click="closeDialog">Batal</v-btn>
        <v-btn color="primary" @click="saveBank">Simpan</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <v-dialog v-model="confirmDialog" max-width="500px">
      <v-card>
        <v-card-title>
          Konfirmasi Hapus
        </v-card-title>
        <v-card-text>
          Apakah Anda yakin ingin menghapus data bank ini?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="cancelDelete">Batal</v-btn>
          <v-btn color="red" dark @click="confirmDelete">Hapus</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      banks: [],
      bankData: [],
      search: '',
      dialog: false,
      isCreate: false,
      editMode: false,
      selectedBankData: {
        id: null,
        bank: '',
        no_rekening: '',
        bank_id: null
      },
      confirmDialog: false,
      headers: [
        { text: 'Foto Bank', value: 'image' },
        { text: 'Nama Bank', value: 'bank' },
        { text: 'No Rekening', value: 'no_rekening' },
        { text: 'Actions', value: 'actions' }
      ]
    };
  },
  mounted() {
    this.fetchBanks();
    this.fetchBankData();
  },
  methods: {
    fetchBanks() {
      this.$axios
        .get('/banks', {
          params: {
            search: this.search
          }
        })
        .then((response) => {
          this.banks = response.data.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    fetchBankData() {
      this.$axios
        .get('/bank-data')
        .then((response) => {
          this.bankData = response.data.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    createBank() {
      console.log(this.selectedBankData)
      this.$axios
        .post('/banks', this.selectedBankData)
        .then((response) => {
          this.fetchBanks();
          this.dialog = false;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    updateBank() {
      this.$axios
        .put(`/banks/${this.selectedBankData.id}`, this.selectedBankData)
        .then((response) => {
          this.fetchBanks();
          this.dialog = false;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    deleteBank(bank) {
      this.selectedBankData = { ...bank };
      this.confirmDialog = true;
    },

    confirmDelete() {
      this.$axios
        .delete(`/banks/${this.selectedBankData.id}`)
        .then((response) => {
          this.fetchBanks();
          this.confirmDialog = false;
        })
        .catch((error) => {
          console.error(error);
          this.confirmDialog = false;
        });
    },

    cancelDelete() {
      this.confirmDialog = false;
    },

    openDialog(bank, isCreate, isEdit) {
      if (isCreate && !isEdit) {
        // Set default values for new bank data
        this.selectedBankData = {
          id: null,
          bank: '',
          no_rekening: '',
          bank_id: null
        };
        this.isCreate = true;
        this.editMode = false;
      } else if (bank && !isCreate && isEdit) {
        this.isCreate = false;
        this.editMode = true;
        this.selectedBankData = { ...bank };
      } else if (bank && !isCreate && !isEdit) {
        this.isCreate = false;
        this.editMode = false;
        this.selectedBankData = { ...bank };
      }
      this.dialog = true;
    },

    saveBank() {
      if (this.editMode) {
        this.updateBank();
      } else {
        this.createBank();
      }
    },

    imageSrc(base64) {
      return `data:image/png;base64,${base64}`;
    },

    closeDialog() {
      this.dialog = false;
    },
   
  }
};
</script>