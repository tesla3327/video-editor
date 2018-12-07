<template>
  <div id="app" @click="handlePlay">
    <video ref="video" controls src="./assets/video2.mp4" />
    <!-- <video ref="video2" controls src="./assets/video2.mp4" /> -->
    <canvas ref="canvas" width="480" height="360"/>
  </div>
</template>

<script>
export default {
  name: 'app',
  mounted() {
    this.ctx = this.$refs.canvas.getContext('2d');
  },
  methods: {
    handlePlay() {
      if (this.$refs.video) {
        this.$refs.video.play();
        this.record();
      }
    },
    record() {
      this.$refs.video.playbackRate = 1;
      this.frames = [];
      this.times = [];
      this.currentTime = 0;
      this.recordFrame();
    },
    recordFrame() {
      const newTime = this.$refs.video.currentTime;
      this.times.push(newTime - this.currentTime);
      this.currentTime = newTime;

      if (this.$refs.video.ended) {
        console.log(this.frames);
        console.log(this.times);
        return;
      }

      const width = this.$refs.video.videoWidth;
      const height = this.$refs.video.videoHeight;

      // Put frame into context
      this.ctx.drawImage(this.$refs.video, 0, 0, width, height);

      // Save frame from context
      this.frames.push(
        this.ctx.getImageData(0, 0, width, height)
      );

      // Call next one
      setTimeout(this.recordFrame, 1000/60);
    }
  }
}
</script>

<style>
video {
  /* display: none; */
  width: 480px;
  height: 360px;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

canvas {
  margin-left: 40px;
  width: 480px;
  height: 360px;
}
</style>
