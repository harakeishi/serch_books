<template>
  <div class="home">
    <input type="number" v-model="isbn"><br>
    <button type="button" @click="postZipcodeRequest">検索！</button>
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
export default {
  name: 'home',
  data () {
    return {
      isbn: null,
      info: null,
      infotext: null
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
        this.info = JSON.parse(JSON.stringify(response.data[0].summary))
        this.infotext = response.data[0].onix.CollateralDetail.TextContent
      })
      .catch(error => {
        alert('検索結果：０件')
      })
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
</style>
