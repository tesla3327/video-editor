<template>
  <div id="app" :class="grabbing && 'grabbing'">
    <div class="videos">
      <video
        v-for="{ url } in videos"
        :key="url"
        ref="video"
        controls
        :src="url"
      />
    </div>
    <canvas ref="canvas" width="480" height="360" />
    <div class="toolbar">
      <div class="controls">
        <button @click="seek(currTime - 10)">&lt;&lt; 10s</button>
        <button @click="seek(currTime - 1)">&lt; 1s</button>
        <button @click="handlePlay">Play</button>
        <button @click="handlePause">Pause</button>
        <button @click="handleStop">Stop</button>
        <button @click="seek(currTime + 1)">1s &gt;</button>
        <button @click="seek(currTime + 10)">10s &gt;&gt;</button>
      </div>
    </div>
    <Timeline
      :keyframes="keyframes"
      :videos="videos"
      :time="currTime"
      :selected="selected"
      :total-length="totalLength"
      @move-left="id => handleMove(id, -1)"
      @move-right="id => handleMove(id, 1)"
      @trim-beginning="handleTrimBeginning"
      @lengthen-beginning="handleLengthenBeginning"
      @trim-end="handleTrimEnd"
      @lengthen-end="handleLengthenEnd"
      @duplicate="handleDuplicate"
      @split="handleSplit"
      @select="handleSelect"
      @delete="handleDelete"
      @start-grab="grabbing = true"
      @end-grab="grabbing = false"
      @add-video="handleAddVideo"
    />
  </div>
</template>

<script>
import Timeline from "./components/Timeline";

let id = 0;
const getId = () => 'id-' + id++;

export default {
  name: "app",

  components: {
    Timeline
  },

  data() {
    return {
      videos: [],
      currTime: 0,
      timeline: [],
      selected: '',
      grabbing: false,
    };
  },

  mounted() {
    this.muteAllVideos();
    this.ctx = this.$refs.canvas.getContext("2d");
  },

  computed: {
    totalLength() {
      return this.keyframes.reduce((prev, next) => prev + next.length, 0);
    },

    keyframes() {
      const frames = [];
      let cumulative = 0;

      for (let i = 0; i < this.timeline.length; i++) {
        const frame = this.timeline[i];
        frames.push({
          ...frame,
          cumulative
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
    seek(time) {
      let targetTime = time;

      if (targetTime < 0) {
        targetTime = 0;
      }

      if (targetTime > this.totalLength) {
        targetTime = this.totalLength;
      }

      // Update current time
      this.currTime = targetTime;

      // Figure out which keyframe we need to be in
      const keyframe = this.keyframes.find(
        ({ length, cumulative }) =>
          targetTime >= cumulative && targetTime < cumulative + length
      );

      // How far into the video we need to be
      const offset = keyframe.start + (targetTime - keyframe.cumulative);

      // Update all the things
      this.$refs.video[keyframe.video].currentTime = offset;
      this.currVideoIndex = keyframe.video;
      this.currentVideo = this.$refs.video[keyframe.video];
    },

    muteAllVideos() {
      if (!this.$refs.video) {
        return;
      }

      // Mute all videos
      this.$refs.video.forEach(video => {
        video.muted = true;
      });
    },

    handleAddVideo(file) {
      // Add the video object
      const url = URL.createObjectURL(file);
      this.videos.push({
        url,
        name: file.name
      });

      // Wait a bit so we can get video duration information
      // (video has to be decoded first??)
      setTimeout(() => {
        // Create region in the timeline
        const index = this.videos.length - 1;
        const length = this.$refs.video[index].duration;
        this.timeline.push({
          id: getId(),
          video: index,
          start: 0,
          length: Math.ceil(length)
        });
        this.muteAllVideos();
      }, 50);
    },

    handleSelect(id) {
      if (this.selected === id) {
        this.selected = -1;
      } else {
        this.selected = id;
      }
    },

    handleDelete(id) {
      const index = this.timeline.findIndex(el => el.id === id);
      this.timeline.splice(index, 1);
    },

    handleDuplicate(id) {
      const index = this.timeline.findIndex(el => el.id === id);

      // Duplicate keyframe
      const keyframe = {
        ...this.timeline[index],
        id: getId(),
      };

      // Insert duplicate
      this.timeline.splice(index, 0, keyframe);
    },

    handleSplit(id) {
      const index = this.timeline.findIndex(el => el.id === id);
      const keyframe = this.timeline[index];

      const first = {
        ...keyframe,
        start: keyframe.start,
        length: Math.ceil(keyframe.length / 2),
        id: getId(),
      };
      const second = {
        ...keyframe,
        start: keyframe.start + length,
        length: Math.floor(keyframe.length / 2),
        id: getId(),
      };

      this.timeline.splice(index, 1, first, second);
    },

    handleTrimBeginning(id) {
      this.timeline = this.timeline.map((keyframe) => {
        if (keyframe.id === id && keyframe.length > 1) {
          return {
            ...keyframe,
            start: keyframe.start + 1,
            length: keyframe.length - 1
          };
        } else {
          return keyframe;
        }
      });
    },

    handleLengthenBeginning(id) {
      this.timeline = this.timeline.map((keyframe) => {
        if (id === keyframe.id && keyframe.start > 0) {
          return {
            ...keyframe,
            start: keyframe.start - 1,
            length: keyframe.length + 1
          };
        } else {
          return keyframe;
        }
      });
    },

    handleTrimEnd(id) {
      this.timeline = this.timeline.map((keyframe) => {
        if (id === keyframe.id && keyframe.length > 1) {
          return {
            ...keyframe,
            length: keyframe.length - 1
          };
        } else {
          return keyframe;
        }
      });
    },

    handleLengthenEnd(id) {
      this.timeline = this.timeline.map((keyframe) => {
        if (id === keyframe.id) {
          return {
            ...keyframe,
            length: keyframe.length + 1
          };
        } else {
          return keyframe;
        }
      });
    },

    // Remove the keyframe from the array and insert it earlier
    handleMove(id, direction) {
      // Create a copy so we don't show intermediate keyframes in the UI
      const temp = [...this.timeline];

      // Grab keyframe
      const index = this.timeline.findIndex(el => el.id === id);
      const keyframe = temp[index];

      // Remove from current position
      temp.splice(index, 1);

      // Move in timeline position
      temp.splice(Math.max(index + direction, 0), 0, keyframe);

      // Set new timeline
      this.timeline = temp;

      // Update selected
      this.selected = '';
      window.focus();
    },

    switchVideo(index, start = 0) {
      if (this.currentVideo) {
        this.currentVideo.pause();
      }

      this.currVideoIndex = index;
      this.currentVideo = this.$refs.video[this.currVideoIndex];

      // Video may be preloaded so we won't need to play it here
      if (this.currentVideo && this.currentVideo.paused) {
        this.currentVideo.currentTime = start;
        this.currentVideo.play();
      }
    },

    preloadVideo(index, start) {
      if (start < 0) {
        return;
      }

      console.log('[PRELOADING]', this.videos[index].name);

      this.$refs.video[index].currentTime = start;
      this.$refs.video[index].play();
    },

    handlePlay() {
      if (!this.currentVideo && this.timeline.length) {
        this.switchVideo(this.timeline[0].video, this.timeline[0].start);
      }

      if (this.currentVideo) {
        this.currentVideo.play();
        this.previousTime = window.performance.now();
        this.drawFrame();
      }
    },

    handlePause() {
      if (this.currentVideo) {
        window.cancelAnimationFrame(this.rAF_ID);
        this.currentVideo.pause();
      }
    },

    handleStop() {
      if (this.currentVideo) {
        window.cancelAnimationFrame(this.rAF_ID);
        this.currentVideo.pause();
        this.currTime = 0;
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
        ({ cumulative }) =>
          cumulative < this.currTime &&
          cumulative >= this.currTime - timeElapsed
      );
      if (keyframe) {
        console.log("[KEYFRAME]", keyframe);
        this.switchVideo(keyframe.video, keyframe.start);
      } else if (this.endTime <= this.currTime) {
        // Stop the video and stop drawing frames
        this.currentVideo.pause();
        return;
      }

      // Check for preloading videos
      const preloadFrame = this.keyframes.find(
        ({ cumulative }) =>
          cumulative < this.currTime + 1 &&
          cumulative >= this.currTime + 1 - timeElapsed
      );
      if (preloadFrame) {
        this.preloadVideo(preloadFrame.video, preloadFrame.start - 1);
      }

      const width = this.currentVideo.videoWidth;
      const height = this.currentVideo.videoHeight;

      // Put frame into context
      this.ctx.drawImage(this.currentVideo, 0, 0, width, height);

      // Draw next frame
      this.rAF_ID = window.requestAnimationFrame(this.drawFrame);
    }
  }
};
</script>

<style lang="scss">
.grabbing * {
  cursor: ew-resize !important;
}

button {
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 14px;
  border: 1px solid #848cab;
  transition: background 0.2s ease-in-out;

  &:hover {
    cursor: pointer;
    background: #ebf0f7;
  }
}

.controls,
.tools {
  display: flex;
  justify-content: center;
  margin: 20px 0;

  * + * {
    margin-left: 10px;
  }
}

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
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

canvas {
  margin: auto;
  width: 480px;
  height: 360px;
  background: black;
}
</style>
