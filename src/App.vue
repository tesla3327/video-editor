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
    <canvas ref="canvas" width="480" height="360"/>
    <Timeline
      :keyframes="keyframes"
      :time="currTime"
      :selected="selected"
      @move-left="index => handleMove(index, -1)"
      @move-right="index => handleMove(index, 1)"
      @select="handleSelect"
    />
  </div>
</template>

<script>
import Timeline from './components/Timeline';

import video1 from './assets/video.mp4';
import video5 from './assets/video5.mp4';

const timeline1 = [
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
];

const timeline2 = [
  {
    video: 0,
    start: 3,
    length: 3,
  },
  {
    video: 0,
    start: 3,
    length: 3
  },
  {
    video: 0,
    start: 3,
    length: 3
  },
  {
    video: 0,
    start: 3,
    length: 3
  },
  {
    video: 0,
    start: 3,
    length: 3
  },
  {
    video: 0,
    start: 3,
    length: 3
  },
];

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
      timeline: timeline1,
      selected: -1,
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
    handleSelect(index) {
      if (this.selected === index) {
        this.selected = -1;
      } else {
        this.selected = index;
      }
    },

    // Remove the keyframe from the array and insert it earlier
    handleMove(index, direction) {
      // Create a copy so we don't show intermediate keyframes in the UI
      const temp = [...this.timeline];

      // Grab keyframe
      const keyframe = temp[index];

      // Remove from current position
      temp.splice(index, 1);

      // Move in timeline position
      temp.splice(Math.max(index + direction, 0), 0, keyframe);

      // Set new timeline
      this.timeline = temp;

      // Update selected
      this.selected = -1;
      window.focus();
    },

    switchVideo(index, start = 0) {
      this.currentVideo.pause();

      if (index !== undefined) {
        this.currVideoIndex = index;
      } else {
        this.currVideoIndex = (this.currVideoIndex + 1) % this.videoUrls.length;
      }

      this.currentVideo = this.$refs.video[this.currVideoIndex];
      this.currentVideo.currentTime = start;
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
        this.switchVideo(keyframe.video, keyframe.start);
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
  margin: 100px auto;
  width: 480px;
  height: 360px;
}
</style>
