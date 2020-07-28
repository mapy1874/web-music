<template>
  <div class="result-container">
    <div class="title-wrap">
      <!-- title -->
      <h2 class="title">{{ $route.query.q }}</h2>
      <span class="sub-title">找到{{count}}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr 
              class="el-table__row" 
              v-for="(item, index) in songList" 
              :key="index"
              @dblclick="playMusic(item.id)"
            >
              <td>{{index+1}}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <!-- song name -->
                    <span>{{item.name}}</span>
                    <span class="iconfont icon-mv" v-if="item.mvid != 0"></span>
                  </div>
                  <!-- secondary name -->
                  <span v-if="item.alias.length!=0">{{item.alias[0]}}</span>
                </div>
              </td>
              <td>{{item.artists[0].name}}</td>
              <td>{{item.album.name}}</td>
              <td>{{item.duration}}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item, index) in playlists" :key="index" @click="toPlaylist(item.id)">
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
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item, index) in mv" :key="index" @click="toMV(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{item.playCount}}</div>
              </div>
              <span class="time">{{item.duration}}</span>
            </div>
            <div class="info-wrap">
              <!-- mv name -->
              <div class="name">{{item.name}}</div>
              <!-- singer name -->
              <div class="singer">{{item.artistName}}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'result',
  data() {
    return {
      activeIndex: 'songs',
      // save queryed songs
      songList: [],
      // save queryed playlist
      playlists: [],
      mv: [],
      count: 0,
    };
  },

  watch: {
    activeIndex() {
      let type = 1;
      // numbers that we get
      let limit = 10;
      switch(this.activeIndex) {
        case 'songs':
          type = 1;
          limit = 10;
          break;
        case "lists":
          type = 1000;
          limit = 10;
          break;
        case "mv":
          type = 1004;
          limit = 8;
          break;
        default:
          break;
      }
      
      axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          limit: limit,
          keywords: this.$route.query.q,
          type: type,
        }
      }).then(resp => {
        console.log(resp);
        if (type==1) {
          // songs
          this.songList = resp.data.result.songs;
          for(let i = 0; i < this.songList.length; i++) {
            let min = parseInt(this.songList[i].duration/1000/60);
            if (min < 10) {
              min  = "0"+min;
            }
            let sec = parseInt(this.songList[i].duration/1000%60);
            if (sec < 10) {
              sec = "0"+sec;
            }
            this.songList[i].duration = min+":"+sec;
          }
          this.count = resp.data.result.songCount;
        } else if (type==1000) {
          // playlist 
          this.playlists = resp.data.result.playlists;
          // total number
          this.count = resp.data.result.playlistCount;
          for (let i = 0; i < this.playlists.length; i++) {
            if(this.playlists[i].playCount>100000) {
              this.playlists[i].playCount = parseInt(this.playlists[i].playCount/10000)+'万';
            }
          }
        } else if (type==1004) {
          this.mv = resp.data.result.mvs;
          this.count = resp.data.result.mvCount;
          console.log(this.mv);
          for (let i = 0; i < this.mv.length; i++) {
            let min = parseInt(this.mv[i].duration/1000/60);
            if(min<10) {
              min = "0"+min;
            }
            let sec = parseInt(this.mv[i].duration/1000%60);
            if (sec < 10){
              sec = "0"+sec;
            }
            this.mv[i].duration = min+":"+sec;
            if(this.mv[i].playCount>100000){
              this.mv[i].playCount = parseInt(this.mv[i].playCount/10000)+'万';
            }
          }
        }
      });
    }
  },
  created() {
    axios({
      url: "https://autumnfish.cn/search",
      method: "get",
      params: {
        limit: 10,
        keywords: this.$route.query.q,
        type: 1,
      }
    }).then(resp => {
      this.songList = resp.data.result.songs;
      for(let i = 0; i < this.songList.length; i++) {
        let min = parseInt(this.songList[i].duration/1000/60);
        if (min < 10) {
          min  = "0"+min;
        }
        let sec = parseInt(this.songList[i].duration/1000%60);
        if (sec < 10) {
          sec = "0"+sec;
        }
        this.songList[i].duration = min+":"+sec;
      }
      this.count = resp.data.result.songCount;
    });
  },

  
  methods: {
    playMusic(id) {
      // console.log(id)
      axios({
        url: "https://autumnfish.cn/song/url",
        method: 'get',
        params: {
          id,
        }
      }).then(resp => {
        let url = resp.data.data[0].url;
        this.$parent.musicUrl =  url;
      });
    },
    toPlaylist(id){
      this.$router.push('/playlist?q='+id);
    },
    toMV(id) {
      this.$router.push("/mv?q="+id);
    },
  }
};
</script>

<style >

</style>
