<template>
  <div
    class="timeline__segment"
    :class="selected && 'timeline__segment--selected'"
    :style="style"
    @click="$emit('select')"
    @keydown.enter="$emit('select')"
    @mousedown="handleBodyMousedown"
    tabindex="0"
  >
    <div
      class="handle handle--left"
      @mousedown.stop="handleStartMousedown"
    />
    <div class="segment-body">
      <span class="video-title">{{ name }}</span>
      <span class="video-range">
        {{`${segment.start}s - ${segment.start + segment.length}s`}}
      </span>
    </div>
    <div
      class="handle handle--right"
      @mousedown.stop="handleEndMousedown"
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
    index: Number,
  },

  data() {
    return {
      endGrabbed: false,
      startGrabbed: false,
      bodyGrabbed: false,
      offset: {
        x: 0,
        y: 0,
      },
      initial: {
        x: 0,
        y: 0,
      }
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

  watch: {
    index() {
      if (!this.rect) {
        return;
      }

      const currRect = this.$el.getBoundingClientRect();

      const diff = {
        x: currRect.x + this.rect.x,
        y: currRect.y - this.rect.y,
      };

      // Update offset
      this.offset.x -= diff.x;
      this.offset.y += diff.y;

      this.initial = {
        x: 0,
        y: 0,
      };

      this.$forceUpdate();
    }
  },

  computed: {
    style() {
      const { x, y } = this.offset;
      return {
        width: `calc(${(this.segment.length / this.totalLength) * 100}% - 2px)`,
        transform: `translate3D(${-x}px, ${-y}px, 0)`,
        'z-index': this.bodyGrabbed ? 1 : 0,
      };
    },
  },

  methods: {
    handleStartMousedown(e) {
      this.$emit('start-grab');
      this.initial.x = e.screenX;
      this.startGrabbed = true;
    },

    handleEndMousedown(e) {
      this.$emit('end-grab');
      this.initial.x = e.screenX;
      this.endGrabbed = true;
    },

    handleBodyMousedown(e) {
      this.initial.x = e.screenX;
      this.initial.y = e.screenY;
      this.bodyGrabbed = true;
    },

    handleMouseMove(e) {
      if (!this.startGrabbed && !this.endGrabbed && !this.bodyGrabbed) {
        return;
      }

      const diff = {
        x: this.initial.x - e.screenX,
        y: this.initial.y - e.screenY,
      };

      const pixelsPerSecond = this.getPixelsPerSecond();

      if (this.startGrabbed) {
        if (diff.x > pixelsPerSecond) {
          this.initial.x = e.screenX;
          this.$emit('lengthen-beginning');
        } else if ((diff.x * -1) > pixelsPerSecond) {
          this.initial.x = e.screenX;
          this.$emit('trim-beginning');
        }
      } else if (this.endGrabbed) {
        if (diff.x > pixelsPerSecond) {
          this.initial.x = e.screenX;
          this.$emit('trim-end');
        } else if ((diff.x * -1) > pixelsPerSecond) {
          this.initial.x = e.screenX;
          this.$emit('lengthen-end');
        }
      } else if (this.bodyGrabbed) {
        this.offset.x = diff.x;
        this.offset.y = diff.y;

        this.rect = this.$el.getBoundingClientRect();

        this.$emit(
          'segment-moved',
          {
            left: this.rect.x + this.offset.x,
            right: this.rect.x + this.offset.x + this.rect.width,
          }
        );
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
      this.initial = {
        x: 0,
        y: 0,
      };
      this.offset = {
        x: 0,
        y: 0,
      };
      this.endGrabbed = false;
      this.startGrabbed = false;
      this.bodyGrabbed = false;
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