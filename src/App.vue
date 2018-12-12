<template>
  <div id="app" :class="grabbing && 'grabbing'">
    <Sidebar
      :selected="selectedObject"
      :type="selected.startsWith('text') ? 'text' : 'video'"
      @change-text="handleChangeText"
      @change-font-size="handleChangeFontSize"
      @change-colour="handleChangeColour"
    />
    <div class="main">
      <div class="videos">
        <video
          v-for="{ url } in videos"
          :key="url"
          ref="video"
          controls
          :src="url"
        />
      </div>
      <div class="viewport">
        <div ref="text" class="text-overlay" />
        <canvas ref="canvas" width="480" height="360" />
      </div>
      <div class="toolbar">
        <div class="controls">
          <VyButtonGroup>
            <VyButton @click="seek(currTime - 10)">&lt;&lt; 10s</VyButton>
            <VyButton @click="seek(currTime - 1)">&lt; 1s</VyButton>
            <VyButton @click="handlePlay">Play</VyButton>
            <VyButton @click="handlePause">Pause</VyButton>
            <VyButton @click="handleStop">Stop</VyButton>
            <VyButton @click="seek(currTime + 1)">1s &gt;</VyButton>
            <VyButton @click="seek(currTime + 10)">10s &gt;&gt;</VyButton>
          </VyButtonGroup>
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
      <Timeline
        :keyframes="textKeyframes"
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
  </div>
</template>

<script>
import { VyButton, VyButtonGroup } from '@vidyard/construction-yard';
import Timeline from "./components/Timeline";
import Sidebar from './components/Sidebar';

let id = 0;
const getId = () => 'id-' + id++;

let colourIndex = 0;
const colours = ['teal', 'turquoise', 'indigo'];
const getColour = () => colours[colourIndex++ % colours.length];

export default {
  name: "app",

  components: {
    Timeline,
    Sidebar,
    VyButton,
    VyButtonGroup,
  },

  data() {
    return {
      videos: [],
      currTime: 0,
      timeline: [],
      textTimeline: [
        {
          id: 'text1',
          text: 'Dance',
          start: 2,
          end: 5,
          colour: 'white',
          fontSize: 24,
        },
      ],
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

    selectedObject() {
      // Could be text or video region
      if (this.selected.startsWith('text')) {
        return this.textTimeline.find(el => el.id === this.selected);
      } else {
        return this.timeline.find(el => el.id === this.selected);
      }
    },

    keyframes() {
      const frames = [];
      let cumulative = 0;

      for (let i = 0; i < this.timeline.length; i++) {
        const frame = this.timeline[i];
        frames.push({
          ...frame,
          name: this.videos[frame.video].name,
          cumulative
        });
        cumulative += frame.length;
      }

      return frames;
    },

    textKeyframes() {
      const frames = [];

      let currTime = 0;
      for (let i = 0; i < this.textTimeline.length; i++) {
        const frame = this.textTimeline[i];

        if (frame.start !== currTime) {
          const length = frame.start - currTime;

          frames.push({
            id: 'blank-' + getId(),
            text: '',
            name: '_',
            length,
            cumulative: currTime,
          });

          currTime += length;
        }

        const length = frame.end - frame.start;
        frames.push({
          ...frame,
          name: frame.text,
          length,
          cumulative: currTime,
        });

        currTime += length;
      }

      // May need another to pad out the end
      if (currTime < this.totalLength) {
        frames.push({
          id: 'blank-' + getId(),
          text: '',
          name: '_',
          length: this.totalLength - currTime,
          cumulative: currTime,
        })
      }

      return frames;
    },

    endTime() {
      const { cumulative, length } = this.keyframes[this.keyframes.length - 1];
      return cumulative + length;
    }
  },

  methods: {
    handleChangeText(text) {
      const textRegion = this.textTimeline.find(el => el.id === this.selected);
      if (textRegion) {
        textRegion.text = text;
        textRegion.name = text;
      }
    },

    handleChangeFontSize(fontSize) {
      const textRegion = this.textTimeline.find(el => el.id === this.selected);
      if (textRegion) {
        textRegion.fontSize = fontSize;
      }
    },

    handleChangeColour(colour) {
      const textRegion = this.textTimeline.find(el => el.id === this.selected);
      if (textRegion) {
        textRegion.colour = colour;
      }
    },

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
          length: Math.ceil(length),
          theme: getColour(),
        });
        this.muteAllVideos();
      }, 50);
    },

    handleSelect(id) {
      if (this.selected === id) {
        this.selected = '';
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
      if (id.startsWith('text')) {
        this.textTimeline = this.textTimeline.map((keyframe) => {
          if (keyframe.id === id && keyframe.start + 1 < keyframe.end) {
            return {
              ...keyframe,
              start: keyframe.start + 1,
            };
          } else {
            return keyframe;
          }
        });
      } else {
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
      }
    },

    handleLengthenBeginning(id) {
      if (id.startsWith('text')) {
        this.textTimeline = this.textTimeline.map((keyframe) => {
          if (id === keyframe.id && keyframe.start > 0) {
            return {
              ...keyframe,
              start: keyframe.start - 1,
            };
          } else {
            return keyframe;
          }
        });
      } else {
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
      }
    },

    handleTrimEnd(id) {
      if (id.startsWith('text')) {
        this.textTimeline = this.textTimeline.map((keyframe) => {
          if (id === keyframe.id && keyframe.start + 1 < keyframe.end) {
            return {
              ...keyframe,
              end: keyframe.end - 1
            };
          } else {
            return keyframe;
          }
        });
      } else {
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
      }
    },

    handleLengthenEnd(id) {
      if (id.startsWith('text')) {
        this.textTimeline = this.textTimeline.map((keyframe) => {
          if (id === keyframe.id) {
            return {
              ...keyframe,
              end: keyframe.end + 1
            };
          } else {
            return keyframe;
          }
        });
      } else {
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
      }
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

    displayTextKeyframe(frame) {
      const el = this.$refs.text;
      el.style.fontSize = `${frame.fontSize}px`;
      el.style.color = frame.colour;
      el.innerText = frame.text;
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

      // Check for text
      const textKeyframe = this.textKeyframes.find(
        ({ cumulative }) =>
          cumulative < this.currTime &&
          cumulative >= this.currTime - timeElapsed
      );
      if (textKeyframe) {
        console.log("[TEXT_KEYFRAME]", textKeyframe);
        this.displayTextKeyframe(textKeyframe);
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
@import '../node_modules/@vidyard/construction-yard/dist/system/system.css';
@import '../node_modules/@vidyard/construction-yard/dist/system/system.utils.scss';

body {
  margin: 0;
  background: #FAFBFF;
}

body, html {
  height: 100%;
  width: 100%;
}

.main {
  width: 100%;
}

.grabbing * {
  cursor: ew-resize !important;
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
  flex-flow: row;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 100%;
  height: 100%;
}

.viewport {
  position: relative;
  margin: auto;
  width: 480px;
  height: 360px;
  background: black;
}

.text-overlay {
  position: absolute;
  padding: 30px;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 74px;
  font-weight: bold;
}

</style>
