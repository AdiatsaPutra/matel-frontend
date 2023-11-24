<template>
  <v-container fluid>
    <v-row>
      <v-col>
        <v-text-field v-model="search" placeholder="Cari member berdasarkan nama" dense solo
          prepend-inner-icon="mdi-magnify"></v-text-field>
      </v-col>
      <v-col>
        <v-btn color="primary" height="40px" dark @click="userChange">
          Download Excel
        </v-btn>
      </v-col>
    </v-row>
    <v-data-table :headers="headers" :items="numberedItems" :search="search">
      <template v-slot:item.subscription_month="{ item }">
        <p class="mt-4 ml-5">
          {{
            getSubscriptionMonthText(
              item.status,
              item.subscription_month,
              item.CreatedAt
            )
          }}
        </p>
      </template>
      <template v-slot:item.start_subscrition="{ item }">
        <p class="mt-4">
          {{
            item.start_subscrition === ""
            ? formatDate(item.CreatedAt)
            : item.status === 2
              ? "-"
              : formatDate(item.start_subscrition)
          }}
        </p>
      </template>
      <template v-slot:item.end_subscription="{ item }">
        <p class="mt-4">
          {{
            item.end_subscription === ""
            ? formatDate(
              new Date(item.CreatedAt).setDate(
                new Date(item.CreatedAt).getDate() + 1
              )
            )
            : item.status === 2
              ? "-"
              : formatDate(item.end_subscription)
          }}
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
          Ubah
        </v-btn>
        <v-btn color="red" height="30px" dark @click="openConfirmModal(item)">
          Hapus
        </v-btn>
      </template>
    </v-data-table>

    <v-dialog v-model="isDetailModalOpen" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">Detail Pengguna</span>
        </v-card-title>
        <div class="px-6">
          <div>Nama Pengguna</div>
          <div class="font-weight-bold">{{ selectedUser.username }}</div>
          <div class="pb-5"></div>
          <div>No HP</div>
          <div class="font-weight-bold">{{ selectedUser.phone }}</div>
          <div class="pb-5"></div>
          <div>Password</div>
          <div class="font-weight-bold">
            {{ selectedUser.password_to_view }}
          </div>
          <div class="pb-5"></div>
          <div>Status</div>
          <div class="font-weight-bold">
            {{
              getStatusText(selectedUser.status, selectedUser.end_subscription)
            }}
          </div>
          <div class="pb-5"></div>
          <div>Email</div>
          <div class="font-weight-bold">
            {{ selectedUser.email }}
          </div>
          <div class="pb-5"></div>
          <div>Device ID</div>
          <div class="font-weight-bold">
            {{ selectedUser.device_id }}
          </div>
          <div class="pb-5"></div>
          <div>Waktu Berlangganan</div>
          <div class="font-weight-bold">
            {{
              getSubscriptionMonthText(
                selectedUser.status,
                selectedUser.subscription_month,
                selectedUser.CreatedAt
              )
            }}
          </div>
          <div class="pb-5"></div>
          <div>Awal Berlangganan</div>
          <div class="font-weight-bold">
            {{ formatDate(selectedUser.start_subscrition) }}
          </div>
          <div class="pb-5"></div>
          <div>Akhir Berlangganan</div>
          <div class="font-weight-bold">
            {{ formatDate(selectedUser.end_subscription) }}
          </div>
          <div class="pb-5"></div>
        </div>
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
          <v-text-field v-model="selectedUser.username" label="Username" readonly outlined></v-text-field>
          <v-select v-model="editUserSubscription" :items="subscriptionOptions" label="Subscription" outlined></v-select>
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
          <v-btn color="primary" dark @click="deleteUser(selectedUser)">Ya</v-btn>
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
        { text: "No HP", value: "phone" },
        { text: "Status", value: "status" },
        { text: "Email", value: "email" },
        { text: "Device ID", value: "device_id" },
        { text: "Waktu Berlangganan", value: "subscription_month" },
        { text: "Mulai Berlangganan", value: "start_subscrition" },
        { text: "Akhir Berlangganan", value: "end_subscription" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      subscriptionOptions: [
        { text: "0 Hari", value: "0" },
        { text: "1 Hari", value: "1" },
        { text: "7 Hari", value: "7" },
        { text: "30 Hari", value: "30" },
        { text: "90 Hari", value: "90" },
        { text: "150 Hari", value: "150" },
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
          },
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
      this.editUserSubscription = user.subscription_month.toString();
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
      if (status === 0) {
        return "Trial";
      } else if (subscriptionEndDate > currentDate) {
        if (status === 0) {
          return "Trial";
        } else if (status === 1) {
          return "Premium";
        } else {
          return "";
        }
      } else {
        return "Expired";
      }
    },
    getSubscriptionMonthText(status, subscriptionMonth, CreatedAt) {
      if (status === 0) {
        const createdAtDate = new Date(CreatedAt);
        const currentDate = new Date();
        const timeDifference = currentDate - createdAtDate;
        const daysDifference = Math.floor(
          timeDifference / (1000 * 60 * 60 * 24)
        );
        const sub = 1 - daysDifference;
        return `${sub} hari`;
      } else {
        if (subscriptionMonth === 1) {
          return "1 hari";
        } else if (subscriptionMonth === 7) {
          return "1 minggu";
        } else if (subscriptionMonth === 30) {
          return "1 bulan";
        } else if (subscriptionMonth > 30 && subscriptionMonth % 30 === 0) {
          const months = subscriptionMonth / 30;
          return `${months} bulan`;
        } else if (subscriptionMonth > 7 && subscriptionMonth % 7 === 0) {
          const weeks = subscriptionMonth / 7;
          return `${weeks} minggu`;
        } else {
          return `0 hari`;
        }
      }
    },
    saveUserChanges() {
      const params = {
        user_id: this.selectedUser.id,
        subscription_month: this.editUserSubscription,
      };
      console.log(params);
      this.$axios
        .post("update-member", params)
        .then((response) => {
          this.closeEditModal();
          this.fetchUser();
        })
        .catch((error) => {
          this.closeEditModal();
          this.fetchUser();
        });
    },
    userChange() {
      this.$axios
        .get("member-change", {
          responseType: 'blob',
        })
        .then((response) => {
          // Create a Blob from the response data
          const blob = new Blob([response.data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });

          // Create a link element
          const link = document.createElement('a');

          // Set the download attribute with the desired file name
          link.download = 'excel_file.xlsx';

          // Create a URL for the Blob and set it as the href attribute of the link
          link.href = window.URL.createObjectURL(blob);

          // Append the link to the document
          document.body.appendChild(link);

          // Programmatically trigger a click on the link to start the download
          link.click();

          // Remove the link from the document
          document.body.removeChild(link);
        })
        .catch((error) => {
          this.closeEditModal();
          this.fetchUser();
        });
    },
    formatDate(date) {
      if (!(date instanceof Date)) {
        date = new Date(date);
      }

      const monthNames = [
        "Januari",
        "Februari",
        "Maret",
        "April",
        "Mei",
        "Juni",
        "Juli",
        "Agustus",
        "September",
        "Oktober",
        "November",
        "December",
      ];

      const day = date.getDate().toString().padStart(2, "0");
      const month = monthNames[date.getMonth()];
      const year = date.getFullYear();

      const formattedDate = `${day} ${month} ${year}`;

      return formattedDate;
    },
  },
  mounted() {
    this.fetchUser();
  },
};
</script>
