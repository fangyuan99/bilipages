<script setup>
import axios from "axios";
import {isObject, useStorage} from '@vueuse/core'
import favoritesList from "../assets/favoritesList.json";
import {computed, onMounted} from "vue";

const api_url = "https://fyapi-fyapi-xtnyzhmgzs.cn-hangzhou.fcapp.run/bili/api";

// onMounted


let curBvid = "";

const deepMerge = (target, source) => {
  if (isObject(target) && isObject(source)) {
    for (const key in source) {
      if (isObject(source[key])) {
        if (!target[key]) Object.assign(target, {[key]: {}});
        deepMerge(target[key], source[key]);
      } else {
        Object.assign(target, {[key]: source[key]});
      }
    }
  }
  return target;
}

var data_ = useStorage('data_', {
      bvid: "",
      interval: null,
      pic: "",
      data: {
        title: "",
        pages: [],
      },
    }, localStorage,
    {
      mergeDefaults: (storageValue, defaults) =>
          deepMerge(defaults, storageValue),
    },);


const get_pages = (bvid) => {
  if (bvid === "") {
    alert("请输入BV号");
    return;
  }
  const pattern = /BV[0-9a-zA-Z]+/;
  const match = bvid.match(pattern);
  if (match) {
    bvid = match[0];
    data_.bvid = bvid;
  } else {
    alert("未能匹配到正确的BV号或AV号");
    return;
  }
  if (bvid === curBvid) {
    alert("数据已存在");
    return;
  }
  set_data({
    bvid,
    interval: null,
    pic: "",
    title: "数据获取中...",
    pages: [],
  })
  //发送请求
  var url = api_url + "?bvid=" + bvid;
  axios
      .get(url)
      .then((res) => {
            if (res.data.code === 0) {
              // console.log(res)
              set_data(res.data.data);
              // data_.data = res.data.data;
            } else {
              set_data({
                bvid: "",
                interval: null,
                pic: "",
                title: `获取失败:${res.data}`,
                pages: [],
              });
            }
          }
      ).catch((err) => {
    alert(`获取失败:${err}`)
  });
};
const set_data = (data) => {
  curBvid = data.bvid;
  data_.value = {
    bvid: data.bvid,
    interval: null,
    pic: data.pic,
    title: data.title,
    pages: data.pages,
  }
};

onMounted(() => {
  //若url中含有bvid参数或者av参数如?BV=或者?av=，则自动获取数据
  const params = new URLSearchParams(window.location.search);
  const bv = params.get("bv");
  const av = params.get("av");
  if (bv) {
    data_.bvid = bv;
    get_pages(bv);
  } else if (av) {
    data_.bvid = av;
    get_pages(av);
  }

});

const toTop = () => {
  // window.scrollTo(0, 0);
  window.scrollTo({
    top: 0,
    behavior: "smooth",
  });
};

const toDown = () => {
  // window.scrollTo(0, document.body.scrollHeight);
  window.scrollTo({
    top: document.body.scrollHeight,
    behavior: "smooth",
  });
};


//实现一个从秒转为格式化时间的函数
const formatSeconds = (value) => {
  var theTime = parseInt(value);// 秒
  var theTime1 = 0;// 分
  var theTime2 = 0;// 小时
  // alert(theTime);
  if (theTime > 60) {
    theTime1 = parseInt(theTime / 60);
    theTime = parseInt(theTime % 60);
    // alert(theTime1+"-"+theTime);
    if (theTime1 > 60) {
      theTime2 = parseInt(theTime1 / 60);
      theTime1 = parseInt(theTime1 % 60);
    }
  }
  var result = "" + parseInt(theTime) + "秒";
  if (theTime1 > 0) {
    result = "" + parseInt(theTime1) + "分" + result;
  }
  if (theTime2 > 0) {
    result = "" + parseInt(theTime2) + "小时" + result;
  }
  return result;
};

const totalTime = computed(() => {
  return data_.value.pages?.reduce((acc, page) => acc + page.duration, 0) ?? 0
})

//通过alert获取用户输入a,b，返回从pages[a]到pages[b]的视频时长总和
const getPTime = () => {
  let a = prompt("请输入起始分p");
  let b = prompt("请输入结束分p");
  if (a === null||a === "") {
    a = 1;
  }
  if (b === null||b === "") {
    b = data_.value.pages.length;
  }

  a = parseInt(a);
  b = parseInt(b);
  if (isNaN(a) || isNaN(b)) {
    alert("请输入数字");
    return;
  }
  if (a < 1 || b < 1 || a > data_.value.pages.length || b > data_.value.pages.length) {
    alert("请输入正确的分p");
    return;
  }
  let sum = 0;
  for (let i = a - 1; i < b; i++) {
    sum += data_.value.pages[i].duration;
  }
  alert(`从P${a}到P${b}的视频时长总和为${formatSeconds(sum)}`);
}

</script>

<template>
  <div class="head">
    <img alt="Bili logo" class="logo" src="../assets/bili.png"/>

    <h1>哔哩哔哩多分p视频检索</h1></div>
  <h2 style="margin: 10px">{{ data_.title }}</h2>
  <div v-if="data_?.pages?.length" @click="getPTime">
    <strong>{{ data_?.pages?.length }}P: </strong><strong>{{ formatSeconds(totalTime) }}</strong>
  </div>
  <div class="operations">
    <h4>
      视频BV号:
      <input v-model="data_.bvid" type="text"/>
    </h4>
    <h4>
      视频关键词:<input v-model="data_.interval" type="text"/>
    </h4>
    <div class="buttons">
      <button style="margin-right: 10px" @click="set_data(favoritesList[0])">一数儿</button>
      <button style="margin-right: 10px" @click="set_data(favoritesList[1])">一化儿</button>
      <button style="margin-right: 10px" @click="get_pages(data_.bvid)">获取数据</button>
    </div>
  </div>


  <hr/>
  <div v-for="item in  data_.pages">
    <ul v-if="data_.interval==null||item.part.indexOf(data_.interval) !== - 1">
      <a
          :href="
          'https://www.bilibili.com/video/' + data_.bvid + '/?p=' + item.page"
          target="_blank"
      ><h4 style="text-align: left">P{{ item.page }}. {{ item.part }}</h4>
        <h4 style="text-align: right">{{ formatSeconds(item.duration) }}</h4>
      </a
      >
    </ul>
  </div>
  <hr/>
  <code class="read-the-docs">
    <h3>本站有什么用?</h3>
    小破站有很多学习视频动辄上百分P,但是没有检索功能,想通过标题找到对应视频很麻烦。<br/>所以我写了这个小工具,可以通过标题检索视频的分p,并且可以跳转到对应的分p。
    <h3>怎么用?</h3>
    1.输入视频的BV号,点击获取数据,就可以获取到视频的标题和分p列表。<br/>
    2.输入想要检索的关键词(没有模糊搜索，注意大小写),就可以检索到对应的分p,并且可以跳转到对应的分p。<br/>
    3.点击标题下面的总时间，可以查询从指定分P到指定分P的时长，比如3P到10P的时长。<br/>
  </code>
  <p class="read-the-docs">fangyuan99</p>
  <button id="TopBtn" style="background: #D4D7DE" @click="toTop">Top↑</button>
  <button id="DownBtn" style="background: #D4D7DE" @click="toDown">Help↓</button>
</template>

<style scoped>
a {
  display: flex;
  justify-content: space-between;
  max-width: 800px;
  margin: 0 auto;
  /* left: 0; */
}

.read-the-docs {
  color: #303133;
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
  bottom: 65px;
  right: 15px;
  position: fixed;
  z-index: 99;
  color: black;
}

#DownBtn {
  bottom: 15px;
  right: 15px;
  position: fixed;
  z-index: 99;
  color: black;
}


.buttons {
  margin: 15px;
}

ul {
  padding: 0;
  margin: 0;
}

.logo {
  height: 38px;
  will-change: filter;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.head {
  display: flex;
  align-items: baseline;
  justify-content: center;
}
</style>
