<script setup>
import axios from "axios";
import { onMounted, reactive } from "vue";
import favoritesList from "../assets/favoritesList.json";

const api_url = "http://bilitotal.fangyuan99.xyz/api";

// onMounted
onMounted(() => {
  // get_pages(data_.bvid);
  data_.data = favoritesList[0];
});

var data_ = reactive({
  data: {
    title: "",
    pages: [],
  },
  interval: "",
  bvid: "",
});
const get_pages = (bvid) => {
  //发送请求
  var url = api_url + "?bvid=" + bvid;
  axios
    .get(url)
    .then((res) => {
      if (res.data.code == 0) data_.data = res.data.data;
      else {
        data_.data = {
          title: res.data,
          pages: [],
        };
      }
    })
    .catch((err) => {
      console.log(err);
    });
};
const set_data = (data) => {
  data_.data = data;
};
</script>

<template>
  <h1>哔哩哔哩多分p视频检索</h1>
  <div class="operations">
    <input type="text" v-model="data_.bvid" />
    <div class="buttons">
      <button @click="set_data(favoritesList[0])">一数儿</button>
      <button @click="set_data(favoritesList[1])">一化儿</button>
      <button @click="get_pages(data_.bvid)">获取数据</button>
    </div>
  </div>
  <h2>{{ data_.data["title"] }}</h2>
  <h4>
    <input type="text" v-model="data_.interval" />
    搜索
  </h4>
  <hr />
  <!-- <p>{{data_["data"]["pages"]}}</p> -->
  <div v-for="(item, index) in data_.data['pages']" :id="item.cid">
    <ul
      v-if="item.part.indexOf(data_.interval) != -1 && data_.interval !== null"
    >
      <a
        :href="
          'https://www.bilibili.com/video/' + data_.bvid + '/?p=' + item.page
        "
        ><h2>{{ index + 1 }}. {{ item.part }}</h2></a
      >
    </ul>
  </div>
  <hr />
  <code class="read-the-docs">
    <h3>本站有什么用?</h3>
    小破站有很多学习视频,但是没有检索功能,所以我写了这个小工具,可以通过标题检索视频的分p,并且可以跳转到对应的分p。
    <h3>怎么用?</h3>
    1.输入视频的BV号,点击获取数据,就可以获取到视频的标题和分p列表。<br />
    2.输入想要检索的关键词,就可以检索到对应的分p,并且可以跳转到对应的分p。<br />
  </code>
  <p class="read-the-docs">fangyuan99</p>
  <span id="TopBtn"><a href="#"> Top↑</a></span>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.operations {
  /* 居中
   */
  margin: 0 auto;
  /* 宽度
   */
}
a {
  text-align: left;
}

#TopBtn {
  background-color: gray;
  bottom: 15px;
  right: 15px;
  position: fixed;
  z-index: 99;
}
#TopBtn a {
  color: #000;
}
.buttons {
  margin: 15px;
}
</style>
