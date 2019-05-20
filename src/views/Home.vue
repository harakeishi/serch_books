<template>
  <div class="home">
    <div class="searchkyeword">
      <input type="text" v-model="searchkyeword" placeholder="作者名か書籍名で検索"><br>
      <input type="button" @click="oncamch()" value="カメラでISBNを読み込む">
      <input type="button" @click="postZipcodeRequest" value="検索！">
    </div>
    <div class="aa" v-if="oncam"></div>
    <div class="cam" v-if="oncam">
      <input type="button" @click="oncamch()" value="閉じる">
      <v-quagga :onDetected="logIt" :readerSize="readerSize" :readerTypes="['ean_reader']" ></v-quagga>　
    </div>
    <div v-if="responsedata">
      <table border="1" v-for="data in responsedata.items">
        <tr>
          <td colspan="2">
            <img :src="data.volumeInfo.imageLinks.thumbnail">
          </td>
        </tr>
        <tr>
          <th colspan="2">タイトル</th>
        </tr>
        <tr>
          <td colspan="2">{{ data.volumeInfo.title }}</td>
        </tr>
        <tr>
          <th>著者</th><th>ISBN-13</th>
        </tr>
        <tr>
          <td>{{ data.volumeInfo.authors[0] }}</td>
          <td>
            <span v-for="isbn in data.volumeInfo.industryIdentifiers" v-if="isbn.type == 'ISBN_13'">
              {{ isbn.identifier }}
            </span>
          </td>
        </tr>
        <tr>
          <th>出版社</th><th>出版日</th>
        </tr>
        <tr>
          <td>{{ data.id }}</td><td>{{ data.volumeInfo.publishedDate }}</td>
        </tr>
        <tr>
          <td colspan="2">{{ data.volumeInfo.description }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import VueQuagga from 'vue-quaggajs'
import bookinfo from '../components/bookinfo'

Vue.use(VueQuagga)

// register component 'v-quagga'
export default {
  name: 'home',
  data () {
    return {
      searchkyeword: null,
      isbn: null,
      readerSize: {
        width: 480,
        height: 220,
      },
      detecteds: [],
      oncam: false,
      responsedata: []
    }
  },
  components: {
    bookinfo
  },
  methods: {
    postZipcodeRequest: function () {
      var url = 'https://www.googleapis.com/books/v1/volumes?q=' + this.searchkyeword
      axios.get(url)
      .then(response => {
          this.responsedata = response.data
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
      if (!this.isbn.match('^978') && !this.isbn.match('^979')) {
        // ISBNかチェックしている。ISBNでなければもう一度撮影
        this.isnb = null
        this.oncam = true
      } else {
        var url = 'https://api.openbd.jp/v1/get?isbn=' + this.isbn
        axios.get(url)
        .then(response => {
          this.searchkyeword = JSON.parse(JSON.stringify(response.data[0].summary.title))
          if (!this.searchkyeword) {
            alert('検索ワードを入力してください！！')
            return
          }
          var url = 'https://www.googleapis.com/books/v1/volumes?q=' + this.searchkyeword
          axios.get(url)
          .then(response => {
            this.responsedata = response.data
          })
          .catch(error => {
            alert('検索結果：０件')
          })
        })
        .catch(error => {
          alert('検索結果：０件')
        })
      }
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
  margin: 10px;
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
