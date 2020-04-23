<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" app clipped>
      <v-list dense>
        <v-subheader>カテゴリー</v-subheader>
        <v-list-item-group v-model="selectedJanre" color="teal" mandatory>
          <v-list-item @click="getAll()" link>
            <v-list-item-content>
              <v-list-item-title>すべて</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-item v-for="j in janres" :key="j.name" @click="getGames(j.id)" link>
            <v-list-item-content>
              <v-list-item-title>{{j.name}}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group>
        <v-divider />
        <v-list-item :href="config.base_url" link>
          <v-list-item-content>
            <v-list-item-title>開発者BLOG</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app color="orange" dark clipped-left>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <div class="d-flex align-center">{{config.site_name}}</div>
    </v-app-bar>

    <v-content>
      <v-container class="my-4">
        <transition name="slide-y-transition" mode="out-in">
          <div :key="selectedJanre">
            <v-alert border="left" color="teal" outlined>
              <div class="title font-weight-bold mb-2">{{ getJanreName(selectedJanre) }}</div>
              <div class="subtitle-1">タイトル数：{{gameCount}}つ</div>
            </v-alert>
            <v-row>
              <v-col
                v-for="g in games"
                :key="g.name"
                v-show="g.isShow"
                cols="12"
                sm="6"
                md="4"
                lg="3"
              >
                <v-card>
                  <v-img :src="getImgSrc(g.url)" :alt="getImgAlt(g.name)" height="180px"></v-img>
                  <v-card-title>{{g.name}}</v-card-title>
                  <v-card-subtitle>{{ getJanreName(g.janre_id) }}</v-card-subtitle>
                  <v-card-text>{{g.description}}</v-card-text>

                  <v-card-actions>
                    <v-btn :href="getUrl(g.url)" outlined dark color="teal">遊ぶ</v-btn>
                  </v-card-actions>
                </v-card>
              </v-col>
            </v-row>
          </div>
        </transition>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import config from "./config.json";
import axios from "axios";

export default {
  name: "App",

  data: () => ({
    gameCount: 0,
    games: [],
    janres: [],
    config: config,
    drawer: false,
    selectedJanre: 0
  }),

  async mounted() {
    let rawDatas = [];
    await axios.get("./datas/informations.json").then(resp => {
      rawDatas = resp.data;
    });
    // isShowの初期値はtrue。jsonファイルに書いてもいいけどゲーム数が膨大になると冗長なのでここで全てにisShow=trueをしておく。
    for (let data of rawDatas) {
      this.gameCount++;
      data.isShow = true;
      this.games.push(data);
    }

    await axios.get("./datas/janres.json").then(resp => {
      this.janres = resp.data;
    });
  },

  methods: {
    getAll: function() {
      let gameCount = 0;
      for (let i = 0; i < this.games.length; i++) {
        gameCount++;
        this.games[i].isShow = true;
      }
      this.gameCount = gameCount;
    },

    getGames: function(janreID) {
      if (this.selectedJanre === janreID) return;

      let gameCount = 0;
      for (let i = 0; i < this.games.length; i++) {
        const isShow = this.games[i].janre_id === janreID;
        this.games[i].isShow = isShow;
        if (isShow) {
          gameCount++;
        }
      }
      this.gameCount = gameCount;
    },

    getJanreName: function(janreID) {
      if (janreID === 0) return "すべて";
      for (const j of this.janres) {
        if (j.id === janreID) {
          return j.name;
        }
      }
    },

    getUrl: function(url) {
      return `${this.config.base_url}/games/${url}`;
    },

    getImgSrc: function(url) {
      return `${this.config.base_url}/games/imgs/${url}.png`;
    },

    getImgAlt: function(name) {
      return `${name}のサムネイル`;
    }
  }
};
</script>

<style>
.v-application {
  font-family: Meiryo, sans-serif;
}
</style>
