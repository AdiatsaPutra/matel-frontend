<template>
  <div>
    <v-text-field
      v-model="search"
      placeholder="Cari Bank"
      solo
      dense
      class="px-3 pt-5"
      prepend-inner-icon="mdi-magnify"
      @input="debouncedFetchBanks"
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
          <template v-slot:item.actions="{ item }">
            <v-btn color="indigo" height="27px" dark @click="openDialog(item, false, false)">
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
          <v-text-field v-model="editedBank.nama_bank" outlined label="Nama Bank"></v-text-field>
          <v-text-field v-model="editedBank.no_rekening" outlined label="No Rekening"></v-text-field>
          <v-text-field v-model="editedBank.foto_bank" outlined label="Foto Bank"></v-text-field>
        </v-card-text>
        <v-card-text v-else>
          <v-text-field v-model="editedBank.nama_bank" readonly outlined label="Nama Bank"></v-text-field>
          <v-text-field v-model="editedBank.no_rekening" readonly outlined label="No Rekening"></v-text-field>
          <v-text-field v-model="editedBank.foto_bank" readonly outlined label="Foto Bank"></v-text-field>
        </v-card-text>
        <v-card-actions v-if="editMode || isCreate">
          <v-spacer></v-spacer>
          <v-btn color="red" dark @click="closeDialog">Batal</v-btn>
          <v-btn color="primary" @click="saveBank">Simpan</v-btn>
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
      search: '',
      dialog: false,
      isCreate: false,
      editMode: false,
      editedBank: {
        nama_bank: '',
        foto_bank: '',
        no_rekening: '',
      },
      headers: [
        { text: 'Foto Bank', value: 'foto_bank' },
        { text: 'Nama Bank', value: 'nama_bank' },
        { text: 'No Rekening', value: 'no_rekening' },
        { text: 'Actions', value: 'actions' }
      ]
    };
  },
  created() {
    this.fetchBanks();
  },
  methods: {
    debouncedFetchBanks: _.debounce(function() {
      this.fetchBanks();
    }, 500),
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
    openDialog(bank, isCreate, isEdit) {
      if (isCreate && !isEdit) {
        this.isCreate = true;
        this.editMode = false;
        this.editedBank = {
          nama_bank: '',
          foto_bank: '',
          no_rekening: '',
        };
      } else if (bank && !isCreate && isEdit) {
        this.isCreate = false;
        this.editMode = true;
        this.editedBank = { ...bank };
      } else if (bank && !isCreate && !isEdit) {
        this.isCreate = false;
        this.editMode = false;
        this.editedBank = { ...bank };
      }
      this.dialog = true;
    },
    saveBank() {
      if (this.editMode) {
        this.$axios
          .put(`/banks/${this.editedBank.id}`, this.editedBank)
          .then((response) => {
            this.fetchBanks();
            this.dialog = false;
          })
          .catch((error) => {
            console.error(error);
          });
      } else {
        this.$axios
          .post('/banks', this.editedBank)
          .then((response) => {
            this.fetchBanks();
            this.dialog = false;
          })
          .catch((error) => {
            console.error(error);
          });
      }
    },
    deleteBank(bank) {
      this.$axios
        .delete(`/banks/${bank.id}`)
        .then((response) => {
          const deletedBank = response.data;
          const index = this.banks.findIndex((bank) => bank.id === deletedBank.id);
          if (index !== -1) {
            this.banks.splice(index, 1);
          }
        })
        .catch((error) => {
          console.error(error);
        });
    },
    closeDialog() {
      this.dialog = false;
    }
  }
};
</script>