<template>
  <div class="play-compontent">
    <div class="audio">
      <audio
        ref="player"
        src='@/assets/123.mp3'
        :autoplay="false"
        @canplay="getDuration"
        @timeupdate="updateTime"
        @ended="audioEnded"
      ></audio>
      <div class="audio-left">
        <svg-icon
          v-if="!audio.playing"
          id="audioPlayer"
          icon-class="play"
          @click="startPlayOrPause"
        />
        <svg-icon
          v-else
          icon-class="pause"
          @click="startPlayOrPause"
        />
      </div>
      <span
        class="audio-length-current"
        id="audioCurTime"
      >{{audio.currentTime}}</span>
      <div
        class="progress-bar-bg"
        id="progressBarBg"
      >
        <span
          id="progressDot"
          :style="`left:${audio.progress * 100}%;`"
          @mousedown="down"
          @touchstart="down"
          @mousemove="move"
          @touchmove="move"
          @mouseup="end"
          @touchend="end"
        ></span>
        <!-- @mousedown="mousedown" -->
        <div
          class="progress-bar"
          :style="`width:${audio.progress * 100}%;`"
          id="progressBar"
        ></div>
      </div>
      <span class="audio-length-total">{{audio.allDuration}}</span>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      audio: {
        playing: false, // 该字段是音频是否处于播放状态的属性
        allDuration: '00:00', // 全部时长
        currentTime: '00:00', // 当前时间
        progress: 0, // 当前进度
        timeid: 0,       //显示剩余时长的计时器
      },
      position: {
        oriOffestLeft: 0, // 移动开始时进度条的点距离进度条的偏移值
        oriX: 0, // 移动开始时的x坐标
        maxLeft: 0, // 向左最大可拖动距离
        maxRight: 136 // 向右最大可拖动距离
      },
      flag: false // 标记是否拖动开始
    }
  },
  mounted () {
  },
  methods: {
    // 获取视频时长
    getDuration () {
      this.audio.allDuration = this.transTime(this.$refs.player.duration)
    },
    // 当前已播放时间
    updateTime (e) {
      this.audio.currentTime = this.transTime(e.target.currentTime)  //获取audio当前播放时间
      this.audio.progress = (e.target.currentTime / this.$refs.player.duration)
    },
    // 控制音频的播放与暂停
    startPlayOrPause () {
      this.audio.playing = !this.audio.playing
      return this.audio.playing ? this.play() : this.pause()
    },
    // 播放结束时
    audioEnded () {
      this.$refs.player.currentTime = 0
      this.audio.progress = 0
      this.audio.playing = false
    },
    // 鼠标拖动
    down (event) {
      var dot = document.getElementById('progressDot')
      if (this.$refs.player.currentTime !== 0) {
        this.flag = true
        this.position.oriOffestLeft = dot.offsetLeft
        this.position.oriX = event.touches ? event.touches[0].clientX : event.clientX; // 要同时适配mousedown和touchstart事件
        this.position.maxLeft = this.position.oriOffestLeft; // 向左最大可拖动距离
        this.position.maxRight = 136 - this.position.oriOffestLeft; // 向右最大可拖动距离
      }
      // 禁止事件冒泡
      if (event && event.stopPropagation) {
        event.stopPropagation();
      } else {
        window.event.cancelBubble = true;
      }
    },
    // 鼠标移动时
    move (event) {
      if (this.flag) {
        var clientX = event.touches ? event.touches[0].clientX : event.clientX; // 要同时适配mousemove和touchmove事件
        var length = clientX - this.position.oriX;
        if (length > this.position.maxRight) {
          length = this.position.maxRight;
        } else if (length < -this.position.maxLeft) {
          length = -this.position.maxLeft;
        }
        var pgsWidth = 136 // 总长度
        var rate = (this.position.oriOffestLeft + length) / pgsWidth;
        this.$refs.player.currentTime = this.$refs.player.duration * rate;
      }
    },
    // 拖拽结束事件
    end () {
      this.flag = false
      if (this.audio.progress === 100) {
        this.$refs.player.currentTime = 0
        this.audio.progress = 0
      }
    },
    // 播放音频
    play () {
      this.$refs.player.play()
    },
    // 暂停音频
    pause () {
      this.$refs.player.pause()
    },
    // 转化播放时间
    transTime (value) {
      var time = "";
      var h = parseInt(value / 3600);
      value %= 3600;
      var m = parseInt(value / 60);
      var s = parseInt(value % 60);
      if (h > 0) {
        time = this.formatTime(h + ":" + m + ":" + s);
      } else {
        time = this.formatTime(m + ":" + s);
      }

      return time;
    },
    formatTime (value) {
      var time = "";
      var s = value.split(':');
      var i = 0;
      for (; i < s.length - 1; i++) {
        time += s[i].length == 1 ? ("0" + s[i]) : s[i];
        time += ":";
      }
      time += s[i].length == 1 ? ("0" + s[i]) : s[i];

      return time;
    }
  }
}
</script>

<style lang="less" scoped>
.play-compontent {
  .audio {
    width: 298px;
    height: 45px;
    background: #f6f6f6ff;
    border-radius: 3px;
    display: flex;
    align-items: center;
    padding: 0 15px;
    margin: 16px auto;
    box-sizing: border-box;
    .audio-left {
      font-size: 32px;
      color: #33beffff;
      margin-right: 6px;
      cursor: pointer;
    }
    .progress-bar-bg {
      flex: 1;
      margin: 0 12px;
      width: 136px;
      height: 5px;
      background: rgba(220, 230, 236, 1);
      border-radius: 3px;
      position: relative;
      cursor: pointer;
      .progress-bar {
        // width: 100%;
        height: 5px;
        background: linear-gradient(
          90deg,
          rgba(98, 201, 255, 1) 0%,
          rgba(51, 153, 255, 1) 100%
        );
        border-radius: 3px;
      }
      span {
        width: 12px;
        height: 12px;
        border-radius: 50%;
        background: rgba(255, 255, 255, 1);
        border: 1px solid rgba(135, 143, 152, 1);
        position: absolute;
        left: 25%;
        top: 50%;
        margin-top: -8px;
        margin-left: -6px;
        cursor: pointer;
      }
    }
  }
}
</style>