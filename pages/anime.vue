<template>
  <div>
    <div>今日のおすすめアニメ診断</div>
    <v-btn @click="rndAnime()">診断する</v-btn>
    <div v-if="rndResult">診断結果：{{ (search = rndResult.anime) }}</div>
    <div v-if="search">
      <v-row
        ><v-col cols="12" lg="4"
          ><v-text-field v-model="search"></v-text-field>
          <v-btn @click="listAnimes()">画像を検索してみる</v-btn></v-col
        ></v-row
      >
    </div>
    <div v-if="apiResult">
      <v-row
        ><v-col
          v-for="(result, index) in apiResult.results"
          :key="index"
          cols="12"
          lg="3"
          sm="6"
          xs="12"
          ><v-card
            >{{ result.title
            }}<v-img :src="result.image_url"></v-img></v-card></v-col
      ></v-row>
    </div>
  </div>
</template>
<script lang="ts">
/* eslint-disable camelcase */
import {
  defineComponent,
  ref,
  useContext,
  watch,
} from '@nuxtjs/composition-api'
interface Anime {
  mal_id: number
  url: string
  image_url: string
  title: string
  airing: boolean
  synopsis: string
  type: 'OVA' | 'TV' | 'Movie' | 'Special'
  episodes: number
  score: number
  start_date: string
  end_date: string
  members: number
  rated: string
}

interface AnimeApiResult {
  request_hash: string
  request_cached: boolean
  request_cache_expiry: number
  results: Anime[]
}

interface AnimeRamResult {
  anime: string
  character: string
  quote: string
}
export default defineComponent({
  setup() {
    // axios：Promiseベースのライブラリ。GETやPOSTを使って鯖にあるデータの取得、更新を行う。
    // useContext：propsなしで親から子にコンポーネントが渡せる。
    const { $axios } = useContext()
    // useContext を呼び出して親から値を受け取り、 axios 変数を宣言（異なる名前のオブジェクトとして名前をつける）
    const apiResult = ref<AnimeApiResult>()
    const search = ref('')

    const listAnimes = async () => {
      apiResult.value = await $axios.$get(
        // ↑axiosのGETメソッド。getの引数のURLに対してGETリクエストを送る。リクエスト後に戻される値はapiResultの中に保存されます。
        'https://api.jikan.moe/v3/search/anime',
        {
          params: {
            q: search,
          },
        } // ↑検索後の結果を表示したURL（？ついたやつ）と一緒
      )
    }
    const rndResult = ref<AnimeRamResult>()

    const rndAnime = async () => {
      rndResult.value = await $axios.$get(
        'https://animechan.vercel.app/api/random'
      )
      //  ↑axios：Promiseを返す。thenやcatchで処理を続けることが可能。
      // 成功時に返ってくるresonseもJSON形式（パースの処理なしに、そのままresponse.dataで使用できる）

      // ↓Fetch API：HTTP通信でリソースを取得する。Fetch APIにより、ページ全体の再読み込みなしに指定URLからデータを取得できます。
      // const animechan = async () => {
      // ramResult.value = await fetch(
      //   'https://animechan.vercel.app/api/random'
      // )
      // .then((response) => response.json())
      // ↑fetchAPIにより返されるResponseオブジェクトは、ただの HTTP レスポンスであり JSON ではありません。
      // 返ってきたresponseにjson()メソッドをつけることでJSONとして使用できます。

      // .then((quote) => console.log(quote))
    }
    watch(search, () => console.log(search.value))
    const clear = () => (search.value = '')

    return {
      apiResult,
      listAnimes,
      search,
      rndResult,
      rndAnime,
      clear,
    }
  },
})
</script>
