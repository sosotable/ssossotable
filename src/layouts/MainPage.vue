<template>
  <q-layout
    view="lHh lpr lFf"
    container
    style="height: 400px"
    class="shadow-2 rounded-borders"
  >
    <q-header bordered class="bg-white text-primary">
      <q-toolbar class="bg-primary glossy text-white">
        <q-btn
          flat
          @click="drawer = !drawer"
          round
          dense
          icon="menu"
          class="q-mr-sm"
        />

        <router-link to="/main/feed" style="margin-right: auto; ">
          <q-avatar>
            <img src="/src/assets/logo.png" />
          </q-avatar>
        </router-link>
        <q-btn flat round dense icon="notifications" />
      </q-toolbar>
    </q-header>
    <q-drawer v-model="drawer" show-if-above :width="200" :breakpoint="400">
      <q-scroll-area
        style="
            height: calc(100% - 150px);
            margin-top: 150px;
            border-right: 1px solid #ddd;
          "
      >
        <q-list padding>
          <q-item
            clickable
            v-ripple
            :active="tab_side === 'contacts'"
            @click="tab_side = 'contacts'"
          >
            <q-item-section avatar>
              <q-icon name="contacts" />
            </q-item-section>
            <q-item-section> 내 정보 </q-item-section>
          </q-item>

          <q-item
            clickable
            v-ripple
            :active="tab_side === 'diversity_1'"
            @click="tab_side = 'diversity_1'"
          >
            <q-item-section avatar>
              <q-icon name="diversity_1" />
            </q-item-section>
            <q-item-section> 친구 </q-item-section>
          </q-item>

          <q-item
            clickable
            v-ripple
            :active="tab_side === 'settings'"
            @click="tab_side = 'settings'"
          >
            <q-item-section avatar>
              <q-icon name="logout" />
            </q-item-section>
            <q-item-section> 설정 </q-item-section>
          </q-item>
          <q-item
            clickable
            v-ripple
            :active="tab_side === 'logout'"
            @click="tab_side = 'logout'"
            class='fixed-bottom'
          >
            <q-item-section avatar>
              <q-icon name="logout" />
            </q-item-section>
            <q-item-section> 로그아웃 </q-item-section>
          </q-item>
        </q-list>
      </q-scroll-area>

      <q-img
        class="absolute-top"
        src="https://cdn.quasar.dev/img/material.png"
        style="height: 150px"
      >
        <div class="absolute-bottom bg-transparent">
          <q-avatar size="56px" class="q-mb-sm">
            <img :src="this.$q.cookies.get('user_image')" />
          </q-avatar>
          <div class="text-weight-bold">{{this.$q.cookies.get('user_nickname')}}</div>
          <div>{{this.$q.cookies.get('user_id')}}</div>
        </div>
      </q-img>
    </q-drawer>
    <q-footer bordered class="bg-white text-primary">
      <q-tabs
        v-model="tab_bottom"
        dense
        align="justify"
        class="bg-primary text-white shadow-2"
        :breakpoint="0"
      >
        <q-tab name="favorite" icon="favorite" />
        <q-tab name="home" icon="home" />
        <q-tab name="language" icon="language" />
      </q-tabs>
    </q-footer>
    <q-page-container>
      <q-page>
        <router-view></router-view>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { useQuasar } from 'quasar';
import MainFeed from 'pages/main/MainFeed.vue';
import MainFriendDiary from 'pages/main/MainFriendDiary.vue';
import MainFriendInfo from 'pages/main/MainFriendInfo.vue';
import MainFriends from 'pages/main/MainFriends.vue';
import MainMyDiary from 'pages/main/MainMyDiary.vue';
import MainMyInfo from 'pages/main/MainMyInfo.vue';
import MainRating from 'pages/main/MainRating.vue';
import MainRecommendation from 'pages/main/MainRecommendation.vue';
import MainRecord from 'pages/main/MainRecord.vue';

export default defineComponent({
  name: 'MainPage',
  setup() {
    const $q = useQuasar();
    const mainFeed = ref < MainFeed > null;
    const mainFriendDiary = ref < MainFriendDiary > null;
    const mainFriendInfo = ref < MainFriendInfo > null;
    const mainFriends = ref < MainFriends > null;
    const mainMyDiary = ref < MainMyDiary > null;
    const mainMyInfo = ref < MainMyInfo > null;
    const mainRating = ref < MainRating > null;
    const mainRecommendation = ref < MainRecommendation > null;
    const mainRecord = ref < MainRecord > null;
    return {
      mainFeed,
      mainFriendDiary,
      mainFriendInfo,
      mainFriends,
      mainMyDiary,
      mainMyInfo,
      mainRating,
      mainRecommendation,
      mainRecord,
      tab_bottom: ref('home'),
      tab_side: ref(null),
      drawer: ref(false),
      $q
    };
  },
  watch: {
    tab_bottom: function () {
      this.tab_side = null;
      switch (this.tab_bottom) {
        case 'home':
          this.$router.push('/main/feed');
          break;
        case 'language':
          this.$router.push('/main/rating');
          break;
        case 'favorite':
          this.$router.push('/main/recommendation');
          break;
      }
    },
    tab_side: function () {
      this.tab_bottom = null;
      switch (this.tab_side) {
        case 'contacts':
          this.$router.push('/main/info/my');
          break;
        case 'diversity_1':
          this.$router.push('/main/friends');
          break;
        case 'settings':
          this.$router.push('/main/setting');
          break;
        case 'logout':
          this.logout()
          break;
      }
    },
  },
  data() {
    return {};
  },
  mounted() {
    //
  },
  methods: {
    side_tab_click: function (event) {
      console.log(event);
    },
    logout: async function() {
      await fetch('http://127.0.0.1:3000/DAO/DELETE', {
        method: 'POST',
        body: new URLSearchParams({
          table: 'sessions',
          where: `user_key in (
          SELECT \`key\`
          FROM user
          WHERE user_id = '${this.$q.cookies.get('user_id')}')`
        }),
      })
      this.$q.cookies.remove('session_key')
      this.$router.push('/')
    }
  },
});
</script>

<style scoped></style>
