<template>
  <div class="timeline-wrapper">
    <div class="timeline">
      <div
        v-for="(segment, index) in keyframes"
        :key="index"
        class="timeline__segment"
        :class="selected === index && 'timeline__segment--selected'"
        :style="{
          width: `calc(${segment.length / total * 100}% - 2px)`
        }"
        @keydown="(e) => handleKeydown(e, index)"
        @click="() => $emit('select', index)"
        @keydown.enter="() => $emit('select', index)"
        tabindex="0"
      >
        <span class="video-title">{{ videos[segment.video].name }}</span>
        <span class="video-range">{{ `${segment.start}s - ${segment.start + segment.length}s` }}</span>
      </div>
      <div
        class="current-time"
        :style="{ left: `${ time / total * 100 }%` }"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'Timeline',

  props: {
    keyframes: Array,
    time: Number,
    selected: Number,
    videos: Array,
  },

  computed: {
    total() {
      return this.keyframes.reduce((prev, next) => prev + next.length, 0);
    }
  },

  methods: {
    handleKeydown(e, index) {
      if (this.selected === -1) {
        return;
      }

      let eventName;

      if (e.key === 'ArrowLeft') {
        if (e.shiftKey) {
          eventName = 'lengthen-beginning';
        } else if (e.metaKey) {
          eventName = 'trim-end';
        } else {
          eventName = 'move-left';
        }
      } else if (e.key === 'ArrowRight') {
        if (e.shiftKey) {
          eventName = 'trim-beginning';
        } else if (e.metaKey) {
          eventName = 'lengthen-end';
        } else {
          eventName = 'move-right';
        }
      } else if (e.code === 'KeyS') {
        eventName = 'split';
      } else if (e.code === 'KeyD' && e.metaKey) {
        eventName = 'duplicate';
      } else if (e.code === 'Delete' || e.code === 'Backspace') {
        eventName = 'delete';
      }

      if (eventName) {
        e.preventDefault();
        e.stopPropagation();
        this.$emit(eventName, index);
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.timeline-wrapper {
  background: #C8CEE3;
  padding: 10px;
}

.timeline {
  position: relative;
  margin: auto;
  width: 100%;
  max-width: 1200px;
  height: 70px;
}

.current-time {
  position: absolute;
  top: 0;
  height: 100%;
  width: 2px;
  background: #7518CC;
  box-shadow: 0 10px 30px 0px rgba(0,0,0,0.8);
}

.timeline__segment {
  display: inline-block;
  box-sizing: border-box;
  padding: 5px 8px;
  margin: 0 1px;
  background: #848CAB;
  color: white;
  height: 100%;
  border-radius: 8px;
  transition: background 0.1s ease-in-out;
}

.video-title {
  display: block;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: -5px;
}

.video-range {
  font-size: 14px;
}

.timeline__segment:hover {
  cursor: pointer;
}

.timeline__segment:focus {
  outline: none;
  box-shadow: 0 0 0 3px #bfc4ff;
}

.timeline__segment--selected {
  background: #6B7394;
}
</style>
