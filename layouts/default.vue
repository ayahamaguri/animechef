<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      color="#3c7d9b"
      dark
      temporary
      width="350"
      :mini-variant="miniVariant"
      :clipped="clipped"
      fixed
      app
    >
      <v-btn class="ma-5" icon @click="drawer = !drawer"
        ><v-icon>mdi-close</v-icon></v-btn
      >
      <v-treeview :items="items" open-all></v-treeview>
    </v-navigation-drawer>
    <v-app-bar
      :clipped-left="clipped"
      fixed
      app
      color="#5b9f8a"
      dark
      elevation="0"
    >
      <div>
        <v-icon
          class="ma-5"
          large
          color="#efebe0"
          @click.stop="drawer = !drawer"
          >mdi-help-circle-outline</v-icon
        >
      </div>
      <!-- <v-btn icon @click.stop="miniVariant = !miniVariant">
        <v-icon>mdi-{{ `chevron-${miniVariant ? 'right' : 'left'}` }}</v-icon>
      </v-btn> -->
      <!-- <v-btn icon @click.stop="clipped = !clipped">
        <v-icon>mdi-application</v-icon>
      </v-btn>
      <v-btn icon @click.stop="fixed = !fixed">
        <v-icon>mdi-minus</v-icon>
      </v-btn> -->
      <v-toolbar-title class="ma-3" v-text="title" />
      <v-spacer />
      <!-- <v-btn icon @click.stop="rightDrawer = !rightDrawer">
        <v-icon>mdi-menu</v-icon>
      </v-btn> -->
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-navigation-drawer v-model="rightDrawer" :right="right" temporary fixed>
      <!-- <v-list>
        <v-list-item @click.native="right = !right">
          <v-list-item-action>
            <v-icon light> mdi-repeat </v-icon>
          </v-list-item-action>
          <v-list-item-title>Switch drawer (click me)</v-list-item-title>
        </v-list-item>
      </v-list> -->
    </v-navigation-drawer>
    <v-footer :absolute="!fixed" app color="#efebe0">
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          id: 1,
          name: 'あそびかた',
          children: [
            {
              id: 2,
              name: ' ①注文するボタンを押す',
            },
            {
              id: 3,
              name: '②気に入ったらシェフを呼ぶ',
            },
            {
              id: 4,
              name: '③好きなタイトルの画像を検索',
            },

            {
              id: 5,
              name: '④お好みの媒体で視聴する',
            },
            {
              id: 6,
              name: '⑤HAPPY！',
            },
          ],
        },
      ],
      // links: [
      //   {
      //     id: 6,
      //     name: 'リンク',
      //     children: [
      //       {
      //         id: 7,
      //         name: 'animechan',
      //         url: 'https://animechan.vercel.app/',
      //       },
      //       {
      //         id: 8,
      //         name: 'jikan',
      //         url: 'https://jikan.moe/',
      //       },
      //     ],
      //   },
      // ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'シェフの気まぐれアニメ',
    }
  },
}
</script>
