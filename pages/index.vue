<template>
  <div class="text-center">
    <div>
      <div>
        <h2 v-if="orderbtn == false">いらっしゃいませ</h2>

        <v-btn
          v-model="orderbtn"
          class="ma-5"
          color="#3c7d9b"
          dark
          @click="
            rndAnime(),
              (orderbtn = true),
              (freeSearchBtn = false),
              (chefBtn = false)
          "
          >注文する</v-btn
        >
      </div>

      <div v-if="rndResult && freeSearchBtn === false">
        <h3>本日のアニメは、{{ rndResult.anime }}でございます</h3>
      </div>
      <div>
        <v-row
          ><v-col v-for="n in 3" :key="n"
            ><v-skeleton-loader
              v-if="loading === true"
              type="image,image"
            ></v-skeleton-loader></v-col
        ></v-row>
      </div>
      <div v-if="apiResult && loading === false">
        <v-row
          ><v-col
            v-for="(result, index) in showAnimes"
            :key="index"
            cols="12"
            lg="4"
            sm="6"
            xs="6"
            ><v-card dark color="#5b9f8a" class="ma-2"
              >{{ result.title
              }}<v-img :src="result.image_url"></v-img></v-card></v-col
        ></v-row>
      </div>
    </div>
    <div v-if="errorCat === true">
      <v-row justify="center"
        ><v-col cols="12" lg="6" sm="8">
          <v-card
            ><v-img :src="'https://http.cat/404.jpg'"></v-img></v-card></v-col
      ></v-row>
    </div>

    <div v-if="rndResult && freeSearchBtn === false">
      <v-btn class="ma-5" color="#efebe0" @click="chefBtn = true"
        ><v-icon left>mdi-bell-outline </v-icon>シェフを呼ぶ</v-btn
      >
    </div>
    <div v-if="chefBtn === true && freeSearchBtn === false">
      本日のシェフ：{{ rndResult.character }}<br />シェフの一言：{{
        rndResult.quote
      }}
    </div>
    <div v-if="rndResult">
      <v-row justify="center"
        ><v-col lg="5">
          <v-text-field
            v-model="search"
            class="mt-5"
            clearable
            color="#5b9f8a"
            placeholder="お好きなアニメを検索できます"
          ></v-text-field>
          <v-btn
            v-model="freeSearchBtn"
            class="md-1"
            color="#3c7d9b"
            dark
            @click="listAnimes(), (freeSearchBtn = true)"
            >画像を検索する</v-btn
          >
        </v-col></v-row
      >
      <div>
        <v-btn class="ma-5" color="#efebe0" @click="callDog()"
          ><v-icon left>mdi-dog</v-icon>犬を呼ぶ</v-btn
        >
      </div>
      <div>
        <v-row justify="center"
          ><v-col cols="12" lg="4" sm="6" xs="6">
            <v-card v-if="dog"
              ><v-img :src="dog.message"></v-img></v-card></v-col
        ></v-row>
      </div>
    </div>
    <div class="mt-10" style="font-family: 'Noto Sans JP', sans-serif">
      <v-chip
        class="ma-1"
        href="https://www.amazon.co.jp/PrimeVideo/"
        color="#101841"
        text-color="white"
        ><v-icon color="#faca7b">mdi-amazon</v-icon> mazon Prime Video</v-chip
      >
      <v-chip
        class="ma-2"
        href="https://www.netflix.com/jp/"
        color="black"
        text-color="red"
        ><strong>NETFLIX</strong></v-chip
      >
      <v-chip
        class="ma-2"
        href="https://anime.dmkt-sp.jp/animestore/"
        color="#da6a38"
        text-color="white"
        ><strong> d </strong>アニメストア
      </v-chip>
      <v-chip
        class="ma-2"
        href="https://www.hulu.jp/display/Anime"
        color="black"
        text-color="#c4d700"
        ><v-icon>mdi-hulu</v-icon><strong>ulu</strong>
      </v-chip>
      <v-chip
        class="ma-2"
        href="https://www.video.unext.jp"
        color="white"
        text-color="primary"
        ><strong>U-NEXT</strong>
      </v-chip>
      <v-chip
        class="ma-2"
        color="white"
        text-color="#d80c18"
        href="https://fod.fujitv.co.jp/"
        ><strong>FOD </strong></v-chip
      >
      <v-chip
        class="ma-2"
        color="black"
        text-color="white"
        href="https://abema.tv/about/premium"
        ><strong> Abema</strong>
      </v-chip>
    </div>
    <div>
      <div class="mt-16">special thanks</div>
      <a href="https://jikan.moe/">{{ 'jikan' }}</a>
      <br />
      <a href="https://animechan.vercel.app/">{{ 'animechan' }}</a>
    </div>
  </div>
</template>
<script lang="ts">
/* eslint-disable camelcase */
import {
  computed,
  defineComponent,
  ref,
  useContext,
  watch,
  // defineAsyncComponent,
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

interface Dog {
  message: string
  status: string
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
      loading.value = true

      apiResult.value = await $axios.$get(
        // ↑axiosのGETメソッド。getの引数のURLに対してGETリクエストを送る。
        // リクエスト後に戻される値はapiResultの中に保存されます。
        'https://api.jikan.moe/v3/search/anime',
        {
          params: {
            q: search,
          },
        } // ↑検索後の結果を表示したURL（？ついたやつ）と一緒
      )
      loading.value = false
    }
    const rndResult = ref<AnimeRamResult>()
    const loading = ref(false)
    const dog = ref<Dog>()
    // async:非同期通信するよ
    // await:promiseの解決(resoleve)するまで待って、変数には結果を代入するよ。thisの進化形
    // axios:非同期処理で（HTTP通信プロトコルを使って）、
    // getとかpostとかヘッダーとかボディとかのお約束でデータをやりとりするよ
    // get:データちょうだい
    const callDog = async () =>
      (dog.value = await $axios.$get('https://dog.ceo/api/breeds/image/random'))

    const rndAnime = async () => {
      loading.value = true
      try {
        // throw console.log('error')
        // HTTP 通信とは
        // HTML Hyper Text Markup language
        // HTTP Hyper Text Transfer Protocol
        // ヘッダー＋ボディを通信先に送信
        // HTTP リクエスト ヘッダー＋ボディ
        //    HTTP メソッド GET POST PUT DELETE
        // HTTP レスポンス ヘッダー＋ボディ
        rndResult.value = await $axios.$get(
          'https://animechan.vercel.app/api/random'
        )
        // await = promiseの結果(resolve)が出るまで待って代入
        // thenで promise a  a.then((b)=>console.log(b)) aはpromise<String>,bはstring

        apiResult.value = await $axios.$get(
          // ↑axiosのGETメソッド。getの引数のURLに対してGETリクエストを送る。
          // リクエスト後に戻される値はapiResultの中に保存されます。
          'https://api.jikan.moe/v3/search/anime',
          {
            params: {
              q: rndResult.value?.anime,
            },
          } // ↑検索後の結果を表示したURL（？ついたやつ）と一緒
        )
      } catch (error) {
        errorCat.value = true
      } finally {
        loading.value = false
      }
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
    }

    watch(search, () => console.log(search.value))
    const clear = () => (search.value = '')
    const showAnimes = computed(() => apiResult.value?.results.slice(0, 3))
    const orderbtn = ref(false)
    const chefBtn = ref(false)
    const freeSearchBtn = ref(false)
    const errorCat = ref(false)

    return {
      apiResult,
      listAnimes,
      search,
      rndResult,
      rndAnime,
      clear,
      showAnimes,
      orderbtn,
      chefBtn,
      freeSearchBtn,
      loading,
      dog,
      callDog,
      errorCat,
    }
  },
})
// <style scoped>
// @import url('https://fonts.googleapis.com/css2?family=Shippori+Mincho&display=swap');
// #app {
//   font-family: 'Shippori Mincho', serif;

//   background-color: #f7c3bf;
// }
// </style>
</script>
