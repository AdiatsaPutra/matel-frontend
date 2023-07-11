<template>
  <div>
    <v-navigation-drawer v-model="drawer" color="primary" dark app>
      <div class="text-h4 font-weight-bold white--text py-5 px-6">Matel</div>
      <v-list color="transparent" density="compact" nav>
        <v-list-item
          v-for="(item, i) in dashboardItem"
          :key="i"
          :to="item.to"
          router
          exact
          :active="activeIndex === i"
          :exact-active-class="exactActiveClass"
          @click="selectItem(i)"
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <v-spacer></v-spacer>
      <template v-slot:append>
        <div class="pa-2">
          <v-btn block text dark @click="logout"> Logout </v-btn>
        </div>
      </template>
    </v-navigation-drawer>
    <v-app-bar fixed app color="white">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-spacer></v-spacer>
      <div class="mr-5">Selamat datang, {{ $auth.user.data.username }}</div>
    </v-app-bar>
  </div>
</template>

<style>
.exact-active-item {
  background-color: rgba(255, 254, 254, 0.051);
}
</style>

<script>
export default {
  name: "NavigationDrawer",
  data() {
    return {
      drawer: true,
      dashboardItem: [
        {
          icon: "mdi-home-outline",
          title: "Home",
          to: "/",
        },
        // {
        //   icon: "mdi-file-upload-outline",
        //   title: "Upload Data",
        //   to: "/upload-data",
        // },
        {
          icon: "mdi-car",
          title: "Data Semua Kendaraan",
          to: "/data-kendaraan",
        },
        {
          icon: "mdi-account-multiple-outline",
          title: "Leasing",
          to: "/leasing-master",
        },
        {
          icon: "mdi-account-outline",
          title: "Member",
          to: "/member",
        },
        {
          icon: "mdi-map-marker-outline",
          title: "Wilayah",
          to: "/wilayah",
        },
        {
          icon: "mdi-bank-outline",
          title: "Info Pembayaran",
          to: "/info-pembayaran",
        },
        // {
        //   icon: "mdi-credit-card-outline",
        //   title: "Nomor Rekening",
        //   to: "/nomor-rekening",
        // },
        // {
        //   icon: "mdi-face-agent",
        //   title: "Customer Support",
        //   to: "/customer-support",
        // },
      ],
      right: true,
      rightDrawer: false,
      exactActiveClass: "exact-active-item",
      activeIndex: null,
    };
  },
  methods: {
    async logout() {
      await this.$auth.logout();
      this.$router.push("/login");
    },
    selectItem(index) {
      this.activeIndex = index;
    },
  },
};
</script>
