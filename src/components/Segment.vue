<template>
  <div
    class="timeline__segment"
    :class="selected && 'timeline__segment--selected'"
    :style="style"
    @click="$emit('select')"
    @keydown.enter="$emit('select')"
    tabindex="0"
  >
    <div
      class="handle handle--left"
      @mousedown="handleStartMousedown"
    />
    <div class="segment-body">
      <span class="video-title">{{ name }}</span>
      <span class="video-range">
        {{`${segment.start}s - ${segment.start + segment.length}s`}}
      </span>
    </div>
    <div
      class="handle handle--right"
      @mousedown="handleEndMousedown"
    />
  </div>
</template>

<script>
export default {
  name: 'Segment',

  props: {
    totalLength: Number,
    segment: Object,
    selected: Boolean,
    name: String,
  },

  data() {
    return {
      endGrabbed: false,
      startGrabbed: false,
    };
  },

  mounted() {
    window.addEventListener('mouseup', this.handleMouseup);
    window.addEventListener('mousemove', this.handleMouseMove);
  },

  destroyed() {
    window.removeEventListener('mouseup', this.handleMouseup);
    window.removeEventListener('mousemove', this.handleMouseMove);
  },

  computed: {
    style() {
      return {
        width: `calc(${(this.segment.length / this.totalLength) * 100}% - 2px)`
      };
    },
  },

  methods: {
    handleStartMousedown(e) {
      this.$emit('start-grab');
      this.initialX = e.screenX;
      this.startGrabbed = true;
    },

    handleEndMousedown(e) {
      this.$emit('end-grab');
      this.initialX = e.screenX;
      this.endGrabbed = true;
    },

    handleMouseMove(e) {
      if (!this.startGrabbed && !this.endGrabbed) {
        return;
      }

      const diff = this.initialX - e.screenX;
      const pixelsPerSecond = this.getPixelsPerSecond();

      if (this.startGrabbed) {
        if (diff > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('lengthen-beginning');
        } else if ((diff * -1) > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('trim-beginning');
        }
      } else if (this.endGrabbed) {
        if (diff > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('trim-end');
        } else if ((diff * -1) > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('lengthen-end');
        }
      }
    },

    // We have to read from the DOM, so it's not great as a computed property
    getPixelsPerSecond() {
      const width = this.$el.clientWidth;
      const { length } = this.segment;
      return width / length;
    },

    handleMouseup() {
      this.$emit('end-grab');
      this.endGrabbed = false;
      this.startGrabbed = false;
    },
  }
}
</script>

<style lang="scss" scoped>
.timeline__segment {
  display: inline-block;
  position: relative;
  box-sizing: border-box;
  padding: 5px 8px;
  margin: 0 1px;
  background: #848cab;
  color: white;
  height: 100%;
  width: 100%;
  border-radius: 8px;
  transition: background 0.1s ease-in-out;

  .handle {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 7px;
    background: #6B7394;
  }

  .handle:hover {
    cursor: ew-resize;
  }

  .handle--left {
    left: 0;
    border-top-left-radius: 8px;
    border-bottom-left-radius: 8px;
  }

  .handle--right {
    right: 0;
    border-top-right-radius: 8px;
    border-bottom-right-radius: 8px;
  }

  .segment-body {
    width: calc(100% - 14px);
    margin-left: 7px;
  }
}

.timeline__segment:hover {
  cursor: grab;
}

.timeline__segment:focus {
  outline: none;
  box-shadow: 0 0 0 3px #bfc4ff;
}

.timeline__segment--selected {
  background: #6b7394;
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

.video-range,
.video-title {
  width: 100%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  user-select: none;
}
</style>