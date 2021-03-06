<template>
  <div id="singer">
    <div class="singer-photo">
      <img v-lazy="imgurl"
           alt="cdlogo">
    </div>
    <div class="header-bar" :style="{background:background}" :class="{dark:isDark}">
      <div class="back-button" @touchend.prevent="hideSinger" @click="hideSinger">
        <div class="back-icon">
          <img src="../assets/icon-back.svg" v-if="!isDark">
          <img src="../assets/icon-back-white.svg" v-if="isDark">
        </div>
        <div class="back-text">
          歌单
        </div>
      </div>
    </div>
    <div id="singer-header" class="header border-1px border-1px-after" v-if="cd!=null">
      <div class="header-blank"></div>
      <div class="header-warp" :style="{background:gradientcolor}">
        <div class="singer-info" :class="{dark:isDark}">
          <h1 class="singer-name">{{cd.dissname}}</h1>
          <p class="singer-fans">{{cd.visitnum | listenCount}}播放 来自：{{cd.nick}}</p>
        </div>
        <div class="play-button" @click="play(0)">
          <img src="../assets/icon-play.png">
        </div>
      </div>
    </div>
    <div class="list" :style="{background:color}" v-if="cd!=null">
      <ul>
        <li class="border-1px border-1px-after" v-for="(item,index) in cd.songlist">
          <div class="music-info" @click="play(index)">
            <div class="music-name" :class="{dark:isDark}">
              {{item.name}}
            </div>
            <div class="music-singer">
              <span v-for="singername in item.singer">{{singername.name}}-</span>
              <span>{{item.subtitle}}</span>
            </div>
          </div>
          <div class="action-button" @touchend.prevent="showMenu(index)" @click="showMenu(index)">
            <img src="../assets/icon-...black.png" alt="">
          </div>
        </li>
      </ul>
      <div class="list-title" :class="{dark:isDark}">
        <p>简介</p>
      </div>
      <div class="singer-brief" :class="{dark:isDark}">
        <p>{{cd.desc}}</p>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">

  export default {
    data () {
      return {
        cd: null,
        opacity: 0,
        menuedIndex: 0,
      }
    },
    methods: {
      hideSinger: function () {
        this.$router.go(-1)
      },
      play: function (index) {
        let list = []
        this.cd.songlist.forEach(item => {
          list.push({
            id: item.id,
            mid: item.mid,
            name: item.name,
            singer: item.singer
          })
        })
        this.$store.commit('setPlayList', {
          index: index,
          list: list
        })
        this.$store.commit('play')
      },
      showMenu: function (num) {
        this.menuedIndex = num
        let that = this
        this.$store.dispatch('notifyActionSheet', {
          menus: {
            'title.noop': this.cd.songlist[num].name + '<br/><span style="color:#666;font-size:12px;">' + this.getSingerStr(this.cd.songlist[num].singer) + '</span>',
            playAsNext: '下一首播放',
            addToPlayList: '添加到播放列表'
          },
          handler: {
            ['cancel'](){
            },
            ['playAsNext'](){
              that.$store.commit('addToPlayListAsNextPlay', {
                id: that.cd.songlist[that.menuedIndex].id,
                mid: that.cd.songlist[that.menuedIndex].mid,
                name: that.cd.songlist[that.menuedIndex].name,
                singer: that.cd.songlist[that.menuedIndex].singer
              })
            },
            ['addToPlayList'](){
              that.$store.commit('addToPlayList', {
                id: that.cd.songlist[that.menuedIndex].id,
                mid: that.cd.songlist[that.menuedIndex].mid,
                name: that.cd.songlist[that.menuedIndex].name,
                singer: that.cd.songlist[that.menuedIndex].singer
              })
            }
          }
        })
      },
      getSingerStr: val => {
        if (typeof val === 'string') {
          return val
        } else if (val instanceof Array) {
          let singer = ''
          val.forEach(item => {
            singer = singer + item.name + ' '
          })
          return singer
        }
      }
    },
    computed: {
      imgurl: function () {
        return this.cd !== null ? this.cd.logo : null
      },
      color: function () {
        if (this.cd !== null) {
          let fixed = '00000' + this.cd.uin.toString(16)
          return '#' + fixed.substr(fixed.length - 6)
        } else {
          return '#ffffff'
        }
      },
      gradientcolor: function () {
        return '-webkit-linear-gradient(top, rgba(' + this.r + ',' + this.g + ',' + this.b + ', 0), ' + this.color + ')'
      },
      isDark: function () {
        let grayLevel = this.r * 0.299 + this.g * 0.587 + this.b * 0.114
        return (grayLevel < 192)
      },
      background: function () {
        return 'rgba(' + this.r + ',' + this.g + ',' + this.b + ',' + this.opacity + ')'
      },
      r: function () {
        return parseInt(this.color.slice(1, 3), 16)
      },
      g: function () {
        return parseInt(this.color.slice(3, 5), 16)
      },
      b: function () {
        return parseInt(this.color.slice(5, 7), 16)
      }
    },
    created: function () {
      this.$store.dispatch('getCdList', this.$route.params.id).then((response) => {
        this.cd = response.data.cdlist[0]
      })

      let that = this
      window.onscroll = function () {
        if (document.getElementById('singer-header')) {
          that.opacity = window.pageYOffset / document.getElementById('singer-header').offsetHeight
        } else {
          that.opacity = 0
        }
      }
    },
    filters: {
      listenCount: num => {
        return Math.round(num / 1000) / 10 + '万'
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  * {
    margin: 0;
    padding: 0;
  }

  .tab-swiper {
    background-color: #fff;
    height: 100px;
  }

  /*border-1px 部分*/
  .border-1px {
    position: relative;
  }

  .border-1px-after:after {
    border-top: 1px solid rgba(255, 255, 255, .15);
    content: ' ';
    display: block;
    width: 100%;
    position: absolute;
    left: 0;
  }

  .border-1px-before:before {
    border-top: 1px solid rgba(255, 255, 255, .15);
    content: ' ';
    display: block;
    width: 100%;
    position: absolute;
    left: 0;
  }

  .border-1px:before {
    top: 0;
  }

  .border-1px:after {
    bottom: 0;
  }

  @media (-webkit-min-device-pixel-ratio: 1.5), (min-device-pixel-ratio: 1.5) {
    .border-1px:after, .border-1px:before {
      -webkit-transform: scaleY(.7);
      -webkit-transform-origin: 0 0;
      transform: scaleY(.7);
    }

    .border-1px:after {
      -webkit-transform-origin: left bottom;
    }
  }

  @media (-webkit-min-device-pixel-ratio: 2), (min-device-pixel-ratio: 2) {
    .border-1px:after, .border-1px:before {
      -webkit-transform: scaleY(.5);
      transform: scaleY(.5);
    }
  }

  #singer {
    display: flex;
    flex-direction: column;
    width: 100%;
    max-width: 68vh;
    min-height: 100%;
    overflow-x: hidden;
    position: absolute;
    top: 0;
    background: #fff;
    z-index: 2;
  }

  .singer-photo {
    position: fixed;
    width: 100%;
    max-width: 68vh;
    height: 100vw;
    max-height: 68vh;
  }

  .singer-photo img {
    width: 100%;
    max-width: 68vh;
    height: 100vw;
    max-height: 68vh;
  }

  .header-bar {
    position: fixed;
    top: 0;
    height: 50px;
    width: 100%;
    max-width: 68vh;
    z-index: 2;
  }

  .header-bar .back-button {
    /*    width:25px;*/
    height: 25px;
    margin: 12.5px;
    margin-left: 5px;
    float: left;
    cursor: pointer;
  }

  .header-bar .back-button .back-icon {
    width: 25px;
    height: 25px;
    float: left;
  }

  .header-bar .back-button .back-icon img {
    width: 25px;
  }

  .header-bar .back-button .back-text {
    line-height: 25px;
    float: left;
  }

  .header {
    width: 100%;
    max-width: 68vh;
    height: 100vw;
    max-height: 68vh;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }

  .header-blank {
    display: flex;
    flex-grow: 1;
  }

  .header-warp {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    /*background: -webkit-linear-gradient(top, rgba(255, 255, 255, 0), rgb(135, 112, 101));*/
  }

  .header-warp .play-button {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    border: 1px solid #eee;
    background: rgba(255, 255, 255, 0.43);
  }

  .header-warp .play-button img {
    display: inline-block;
    width: 25px;
    height: 25px;
  }

  .list {
    /*background: rgb(135, 112, 101);*/
    z-index: 1;
    padding-bottom: 50px;
  }

  .list .list-title {
    text-align: center;
    line-height: 40px;
  }

  .list ul {
    list-style: none;
    padding-left: 10px;
  }

  .list ul li {
    width: 100%;
    display: flex;
    display: -webkit-flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    height: 60px;
    cursor: pointer;
  }

  .list ul li .music-info {
    display: flex;
    display: -webkit-flex;
    flex-direction: column;
    flex-grow: 1;
    overflow: hidden;
  }

  .list ul li .music-info .music-name {
    width: 100%;
    color: #000;
    line-height: 22px;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }

  .dark {
    color: #fff !important;
  }

  .list ul li .music-info .music-singer {
    color: #aaa;
    font-size: 14px;
    line-height: 22px;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }

  .list ul li .action-button {
    width: 25px;
    height: 25px;
    margin-right: 10px;
  }

  .list ul li .action-button img {
    width: 25px;
    height: 25px;
  }

  .list .singer-brief {
    font-size: 14px;
    padding: 10px;
  }
</style>
