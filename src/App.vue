<template>
  <div id="app" @click="handlePlay">
    <div class="videos">
      <video
        v-for="url in videoUrls"
        :key="url"
        ref="video"
        controls
        :src="url"
      />
    </div>
    Canvas:
    <canvas ref="canvas" width="480" height="360"/>
  </div>
</template>

<script>
import video1 from './assets/video.mp4';
import video5 from './assets/video5.mp4';

export default {
  name: 'app',

  data() {
    return {
      videoUrls: [
        video1,
        video5,
      ],
      currentVideoIndex: 0,
    };
  },

  mounted() {
    // Mute all videos
    this.$refs.video.forEach(video => {
      video.muted = true;
    });

    this.ctx = this.$refs.canvas.getContext('2d');

    // Switch current video every few seconds
    setInterval(this.switchVideo, 3000);
  },

  computed: {
    currentVideo() {
      return this.$refs.video[this.currentVideoIndex];
    }
  },

  methods: {
    switchVideo() {
      this.currentVideo.pause();
      this.currentVideoIndex = (this.currentVideoIndex + 1) % this.videoUrls.length;
      this.currentVideo.play();
    },
    handlePlay() {
      if (this.currentVideo) {
        this.currentVideo.play();
        this.drawFrame();
      }
    },
    drawFrame() {
      const width = this.currentVideo.videoWidth;
      const height = this.currentVideo.videoHeight;

      // Put frame into context
      this.ctx.drawImage(this.currentVideo, 0, 0, width, height);

      // Draw next frame
      this.rAF_ID = window.requestAnimationFrame(this.drawFrame);
    },
  }
}
</script>

<style>
video {
  /* display: none; */
  width: 480px;
  height: 360px;
}

.videos {
  margin-bottom: 50px;
}

#app {
  display: flex;
  flex-flow: column;
  justify-content: center;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

canvas {
  margin: auto;
  width: 480px;
  height: 360px;
}
</style>
