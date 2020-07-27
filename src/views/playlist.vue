<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
         <!-- cover -->
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <!-- name -->
        <p class="title">{{ playlist.name }}</p>
        <div class="author-wrap">
          <!-- avatar -->
          <img class="avatar" :src="playlist.creator.avatarUrl" alt="" />
          <span class="name">{{playlist.creator.nickname}}</span>
          <span class="time">{{playlist.createTime}} 创建</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item, index) in playlist.tags" :key="index">{{item}}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <span class="desc"
            >{{playlist.description}}</span
          >
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item, index) in playlist.tracks" :key="index">
              <td>{{index+1}}</td>
              <td>
                <div class="img-wrap" @click="playMusic(item.id)">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <!-- mv icon -->
                    <span v-if="item.mv != 0" @click="toMV(item.mv)" class="iconfont icon-mv"></span>
                  </div>
                  <span>{{item.subTitle}}</span>
                </div>
              </td>
              <td>{{item.ar[0].name}}</td>
              <td>{{item.al.name}}</td>
              <td>{{ item.dt }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="评论(66)" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">精彩评论<span class="number">({{hotCount}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item, index) in hotComment" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <!-- nickname -->
                  <span class="name">{{item.user.nickname}}</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <!-- reply -->
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">2020-02-12 17:26:11</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{total}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item, index) in comments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <div class="re-content" v-if="item.heReplied.length!=0">
                  <span class="name">{{item.heReplied[0].user.nickname}}</span>
                  <span class="comment">{{item.heReplied[0].user.content}}</span>
                </div>
                <div class="date">2020-02-12 17:26:11</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 分页器 -->
        <el-pagination
          @current-change="handleCurrentChange"
          background
          layout="prev, pager, next"
          :total="total"
          :current-page="page"
          :page-size="5"
        >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment';

export default {
  name: 'playlist',

  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      playlist:{},
      hotComment: [],
      comments: [],
      hotCount: 0,
    };
  },
  created() {
    // get the detail of the playlist
    axios({
      url: 'https://autumnfish.cn/playlist/detail',
      method: 'get',
      params: {
        id: this.$route.query.q,
      },
    }).then(resp => {
      this.playlist = resp.data.playlist;
      this.playlist.createTime = moment(this.playlist.createTime).format('YYYY-MM-DD');
      
      // deal with time of the song
      for (let i = 0; i < this.playlist.tracks.length; i++) {
        const element = this.playlist.tracks[i];
        let min = parseInt(element.dt/1000/60);
        if (min < 10) {
          min  = "0"+min;
        }
        let sec = parseInt(element.dt/1000%60);
        if (sec < 10) {
          sec = "0"+sec;
        }
        element.dt = min+":"+sec;      
      }
    });

    // get the comments
    axios({
      url: "https://autumnfish.cn/comment/hot",
      method:"get",
      params:{
        id: this.$route.query.q,
        type: 2,
      },
    }).then(resp => {
      this.hotComment = resp.data.hotComments;
      this.hotCount = resp.data.total;
    });

    // get newest comments
    axios({
      url: "https://autumnfish.cn/comment/playlist",
      method: 'get',
      params: {
        id: this.$route.query.q,
        limit: 10,
        offset: 0,
      },
    }).then(resp => {
      // console.log(resp);
      this.total =  resp.data.total;
      this.comments = resp.data.comments;
    })
  },
  methods: {
    handleCurrentChange(val) {
      // get the page number
      this.page = val;
      // get the data again
      axios({
        url: "https://autumnfish.cn/comment/playlist",
        method: 'get',
        params: {
          id: this.$route.query.q,
          limit: 10,
          offset: (this.page-1)*10,
        },
      }).then(resp => {
        // console.log(resp);
        this.total =  resp.data.total;
        this.comments = resp.data.comments;
      })
    },

    toMV(id) {
      this.$router.push("/mv?id="+id);
    },
    playMusic(id) {
      axios({
        url: "https://autumnfish.cn/song/url",
        method: 'get',
        params: {
          id,
        }
      }).then(resp => {
        let url = resp.data.data[0].url;
        this.$parent.musicUrl = url;
      });
    },
  }
};
</script>

<style >

</style>