<template>
  <div class="playlists-container">
    <!-- 同步 -->
    <div class="top-card">
      <div class="icon-wrap">
        <img :src="topList.coverImgUrl" alt="" />
      </div>
      <div class="content-wrap">
        <div class="tag">精品歌单</div>
        <div class="title">
          {{ topList.name }}
        </div>
        <div class="info">
          {{ topList.description }}
        </div>
      </div>
      <img :src="topList.coverImgUrl" alt="" class="bg" />
      <div class="bg-mask"></div>
    </div>
    <div class="tab-container">
      <!-- tab栏 顶部 -->
      <div class="tab-bar">
        <span class="item" @click="tag='全部'" :class="{active:tag=='全部'}">全部</span>
        <span class="item" @click="tag='欧美'" :class="{active:tag=='欧美'}">欧美</span>
        <span class="item" @click="tag='华语'" :class="{active:tag=='华语'}">华语</span>
        <span class="item" @click="tag='流行'" :class="{active:tag=='流行'}">流行</span>
        <span class="item" @click="tag='说唱'" :class="{active:tag=='说唱'}">说唱</span>
        <span class="item" @click="tag='摇滚'" :class="{active:tag=='摇滚'}">摇滚</span>
        <span class="item" @click="tag='民谣'" :class="{active:tag=='民谣'}">民谣</span>
        <span class="item" @click="tag='电子'" :class="{active:tag=='电子'}">电子</span>
        <span class="item" @click="tag='轻音乐'" :class="{active:tag=='轻音乐'}">轻音乐</span>
        <span class="item" @click="tag='影视原声'" :class="{active:tag=='影视原声'}">影视原声</span>
        <span class="item" @click="tag='ACG'" :class="{active:tag=='ACG'}">ACG</span>
        <span class="item" @click="tag='怀旧'" :class="{active:tag=='怀旧'}">怀旧</span>
        <span class="item" @click="tag='治愈'" :class="{active:tag=='治愈'}">治愈</span>
        <span class="item" @click="tag='旅行'" :class="{active:tag=='旅行'}">旅行</span>
      </div>
    <!-- tab的内容区域 -->
      <div class="tab-content">
        <div class="items">
          <div class="item" v-for="(item, index) in list" :key="index">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{item.playCount}}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{item.name}}</p>
          </div>
        </div>
      </div>
    </div>
    <!-- 分页器 
        total 总条数
        current-page 当前页
        page-size number of data in each page
    -->
    <el-pagination
      @current-change="handleCurrentChange"
      background
      layout="prev, pager, next"
      :total="total"
      :current-page="page"
      :page-size="10"
    >
    </el-pagination>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'recommend',
  data() {
    return {
      // 总条数
      total:0,
      // 页码
      page:1,
      // top recommendated list
      topList: {},
      list: [],
      tag: '全部',
    };
  },
  // 侦听器
  watch: {
    tag() {
      this.topData();
      this.listData();
      this.page = 1;
    }
  },

  created() {
    this.topData();
    this.listData();
  },
  methods: {
    // refractor method 1 
    topData() {
      axios({
        url: "https://autumnfish.cn/top/playlist/highquality",
        method: "get",
        params: {
          limit: 1,
          cat: this.tag,
        },
      }).then(resp => {
        this.topList = resp.data.playlists[0];
      });
    },

    // refractor method 2
    listData() {
      axios({
        url: "https://autumnfish.cn/top/playlist/",
        method: "get",
        params: {
          limit: 10,
          offset: (this.page-1)*10,
          cat: this.tag,
        },
      }).then(resp => {
        this.total = resp.data.total;
        this.list = resp.data.playlists;
      });
    },
    handleCurrentChange(val) {
      this.page = val;
      // refetch the data
      this.listData();
    }
  }
};
</script>

<style >

</style>
