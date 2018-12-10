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
    <Timeline :keyframes="keyframes" :time="currTime" />
  </div>
</template>

<script>
import Timeline from './components/Timeline';

import video1 from './assets/video.mp4';
import video5 from './assets/video5.mp4';

export default {
  name: 'app',

  components: {
    Timeline,
  },

  data() {
    return {
      videoUrls: [
        video1,
        video5,
      ],
      currTime: 0,
      timeline: [
        {
          video: 0,
          start: 0,
          length: 3
        },
        {
          video: 1,
          start: 0,
          length: 3
        },
        {
          video: 0,
          start: 0,
          length: 5
        },
        {
          video: 1,
          start: 0,
          length: 10
        },
        {
          video: 0,
          start: 0,
          length: 10
        },
      ]
    };
  },

  mounted() {
    // Mute all videos
    this.$refs.video.forEach(video => {
      video.muted = true;
    });

    // Setup selected video
    this.currVideoIndex = 0;
    this.currentVideo = this.$refs.video[0];

    this.ctx = this.$refs.canvas.getContext('2d');
  },

  computed: {
    keyframes() {
      const frames = [];
      let cumulative = 0;

      for (let i = 0; i < this.timeline.length; i++) {
        const frame = this.timeline[i];
        frames.push({
          ...frame,
          cumulative,
        });
        cumulative += frame.length;
      }

      return frames;
    },

    endTime() {
      const { cumulative, length } = this.keyframes[this.keyframes.length - 1];
      return cumulative + length;
    }
  },

  methods: {
    switchVideo(index) {
      this.currentVideo.pause();

      if (index !== undefined) {
        this.currVideoIndex = index;
      } else {
        this.currVideoIndex = (this.currVideoIndex + 1) % this.videoUrls.length;
      }

      this.currentVideo = this.$refs.video[this.currVideoIndex];
      this.currentVideo.play();
    },

    handlePlay() {
      if (this.currentVideo) {
        this.currentVideo.play();
        this.previousTime = window.performance.now();
        this.drawFrame();
      }
    },

    drawFrame() {
      // Update time
      const currTime = window.performance.now();
      const timeElapsed = (currTime - this.previousTime) / 1000;
      this.currTime += timeElapsed;
      this.previousTime = currTime;

      // Check if we need to switch the video
      const keyframe = this.keyframes.find(
        ({ cumulative }) => cumulative < this.currTime &&
                            cumulative >= (this.currTime - timeElapsed)
      );

      if (keyframe) {
        console.log('[KEYFRAME]', keyframe);
        this.switchVideo(keyframe.video);
      } else if (this.endTime <= this.currTime) {
        // Stop the video and stop drawing frames
        this.currentVideo.pause();
        return;
      }

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
  display: none;
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
  color: #2c3e50;
  margin-top: 60px;
}

canvas {
  margin: auto;
  width: 480px;
  height: 360px;
}
</style>