<template>
  <v-container fluid>
    <v-row>
      <v-col>
        <v-text-field
          v-model="search"
          placeholder="Cari member berdasarkan nama"
          dense
          solo
          prepend-inner-icon="mdi-magnify"
        ></v-text-field>
      </v-col>
    </v-row>
    <v-data-table :headers="headers" :items="numberedItems">
      <template v-slot:item.subscription_month="{ item }">
        <p class="mt-4 mr-5 text-center">
          {{ getSubscriptionMonthText(item.subscription_month) }}
        </p>
      </template>
      <template v-slot:item.start_subscrition="{ item }">
        <p class="mt-4 mr-5 text-center">
          {{ item.start_subscrition === "" ? "-" : item.start_subscrition }}
        </p>
      </template>
      <template v-slot:item.end_subscription="{ item }">
        <p class="mt-4 mr-5 text-center">
          {{ item.end_subscription === "" ? "-" : item.end_subscription }}
        </p>
      </template>
      <template v-slot:item.status="{ item }">
        {{ getStatusText(item.status, item.end_subscription) }}
      </template>
      <template v-slot:item.actions="{ item }">
        <v-btn color="primary" height="30px" dark @click="openDetailModal(item)">
          Detail
        </v-btn>
        <v-btn color="primary" height="30px" outlined dark @click="openEditModal(item)">
          Ubah Subscription
        </v-btn>
        <v-btn color="red" height="30px" dark @click="openConfirmModal(item)"> Hapus </v-btn>
      </template>
    </v-data-table>

    <v-dialog v-model="isDetailModalOpen" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">Detail Pengguna</span>
        </v-card-title>
        <v-card-text>
          <v-text-field
            v-model="selectedUser.username"
            label="Username"
            readonly
            outlined
          ></v-text-field>
          <v-text-field
            v-model="selectedUser.email"
            label="Email"
            readonly
            outlined
          ></v-text-field>
          <v-text-field
            :value="getStatusText(selectedUser.status, selectedUser.end_subscription)"
            label="Status"
            readonly
            outlined
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" dark @click="closeDetailModal">Tutup</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="isEditModalOpen" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">Ubah Subscription Pengguna</span>
        </v-card-title>
        <v-card-text>
          <v-text-field
            v-model="selectedUser.username"
            label="Username"
            readonly
            outlined
          ></v-text-field>
          <v-select
            v-model="editUserSubscription"
            :items="subscriptionOptions"
            label="Subscription"
            outlined
          ></v-select>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" dark @click="saveUserChanges">Simpan</v-btn>
          <v-btn color="red" dark @click="closeEditModal">Batal</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="isConfirmModalOpen" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">Konfirmasi Hapus Pengguna</span>
        </v-card-title>
        <v-card-text>
          Apakah Anda yakin ingin menghapus pengguna ini?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" dark @click="deleteUser(selectedUser)"
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
      users: [],
      search: "",
      isDetailModalOpen: false,
      isEditModalOpen: false,
      isConfirmModalOpen: false,
      selectedUser: {
        id: 0,
        username: "",
        status: 0,
      },
      editUserSubscription: 1,
      headers: [
        { text: "No", value: "no" },
        { text: "Nama", value: "username" },
        { text: "Email", value: "email" },
        { text: "Status", value: "status" },
        { text: "Waktu Berlangganan", value: "subscription_month" },
        { text: "Mulai Berlangganan", value: "start_subscrition" },
        { text: "Akhir Berlangganan", value: "end_subscription" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      subscriptionOptions: [
        { text: "1 Bulan", value: "1" },
        { text: "3 Bulan", value: "3" },
        { text: "6 Bulan", value: "6" },
        { text: "12 Bulan", value: "12" },
      ],
    };
  },
  computed: {
    numberedItems() {
      return this.users.map((item, index) => ({
        ...item,
        no: index + 1,
      }));
    },
  },
  methods: {
    fetchUser() {
      this.$axios
        .get("member", {
          params: {
            search: this.search,
          }
        })
        .then((response) => {
          if (response.data.data === null) {
            this.users = [];
          } else {
            this.users = response.data.data;
          }
        })
        .catch((error) => {
          console.error(error);
        });
    },
    openDetailModal(user) {
      this.selectedUser = user;
      this.isDetailModalOpen = true;
    },
    closeDetailModal() {
      this.isDetailModalOpen = false;
    },
    openEditModal(user) {
      this.selectedUser = user;
      this.isEditModalOpen = true;
    },
    closeEditModal() {
      this.isEditModalOpen = false;
    },
    openConfirmModal(user) {
      this.selectedUser = user;
      this.isConfirmModalOpen = true;
    },
    closeConfirmModal() {
      this.isConfirmModalOpen = false;
    },
    deleteUser(user) {
      this.$axios
        .delete(`delete-member/${this.selectedUser.id}`)
        .then((response) => {
            this.closeEditModal();
            this.fetchUser();
        })
        .catch((error) => {
          console.error(error);
        });
      this.closeConfirmModal();
    },
    getStatusText(status, end) {
      const currentDate = new Date();
      const subscriptionEndDate = new Date(end);
      if (subscriptionEndDate > currentDate) {
        if (status === 0) {
          return "Trial"
        } else if (status === 1) {
          return "Premium"
        } else {
          return ""
        }
      } else {
        return "Expired"
      }
    },
    getSubscriptionMonthText(subscriptionMonth) {
      if (subscriptionMonth === 0) {
        return "-"
      } else {
        return `${subscriptionMonth} Bulan`
      }
    },
    saveUserChanges() {
      const params = {
        user_id: this.selectedUser.id,
        subscription_month: this.editUserSubscription,
      };

      this.$axios
        .post('update-member', params)
        .then((response) => {
          this.closeEditModal();
          this.fetchUser()
        })
        .catch((error) => {
          this.closeEditModal();
          this.fetchUser()
        });
    },
  },
  mounted() {
    this.fetchUser();
  },
};
</script>