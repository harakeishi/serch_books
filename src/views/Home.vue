<template>
  <div class="home">
    <input type="number" v-model="isbn"><br>
    <input type="button" @click="oncamch()" value="カメラで読み込む">
    <button type="button" @click="postZipcodeRequest">検索！</button>
    <div class="aa" v-if="oncam"></div>
    <div class="cam" v-if="oncam">
      <input type="button" @click="oncamch()" value="閉じる">
      <v-quagga :onDetected="logIt" :readerSize="readerSize" aspectRatio="min: 1, max:1" :readerTypes="['ean_reader']" ></v-quagga>　
    </div>
    <div v-if="info">
      <table border="1">
        <tr>
          <td colspan="2">
            <img :src="info.cover">
          </td>
        </tr>
        <tr>
          <th colspan="2">タイトル</th>
        </tr>
        <tr>
          <td colspan="2">{{ info.title }}</td>
        </tr>
        <tr>
          <th>著者</th><th>ISBN-10</th>
        </tr>
        <tr>
          <td>{{ info.author }}</td><td>{{ info.isbn }}</td>
        </tr>
        <tr>
          <th>出版社</th><th>出版日</th>
        </tr>
        <tr>
          <td>{{ info.publisher }}</td><td>{{ info.pubdate }}</td>
        </tr>
        <tr v-if="infotext">
          <td colspan="2">{{ infotext[0].Text }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import VueQuagga from 'vue-quaggajs';
Vue.use(VueQuagga);

// register component 'v-quagga'
export default {
  name: 'home',
  data () {
    return {
      isbn: null,
      info: null,
      infotext: null,
      readerSize: {
        width: 480,
        height: 220,
      },
      detecteds: [],
      oncam: false,
      aspectRatio: {
        min: 1,
        max: 1
      }
    }
  },
  components: {
  },
  methods: {
    postZipcodeRequest: function () {
      if (!this.isbn) {
        alert('ISBNを入力してください！！')
        return
      }
      var url = 'https://api.openbd.jp/v1/get?isbn=' + this.isbn
      axios.get(url)
      .then(response => {
        console.log(response)
        this.info = JSON.parse(JSON.stringify(response.data[0].summary))
        this.infotext = response.data[0].onix.CollateralDetail.TextContent
      })
      .catch(error => {
        alert('検索結果：０件')
      })
    },
    oncamch () {
      if (this.oncam) {
        this.oncam = false
      } else {
        this.oncam = true
      }
    },
    logIt (data) {
      this.oncam = false
      this.isbn = data.codeResult.code
    }
  }
}
</script>

<style>
th{
  background-color: rgb(125, 214, 212);
}
table{
  width: 100%;
  white-space: normal;
}
.cam{
  width: 350px;
  height: 350px;
  position: absolute;
  margin: 0 auto;
}
video{
  width: 350px;
  height: 350px;
}
canvas{
  width: 350px;
  height: 350px;
}
.aa{
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.48);
}
</style>
