<template>
  <div
    class="timeline__segment"
    :class="[
      selected && 'timeline__segment--selected',
      colour,
      hidden && 'hidden'
    ]"
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
      <span class="video-title" :class="type">{{ name.trim() || '&nbsp;' }}</span>
      <span v-if="type === 'video'" class="video-range">
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
    colour: String,
    hidden: Boolean,
    type: String,
  },

  data() {
    return {
      endGrabbed: false,
      startGrabbed: false,
      bodyGrabbed: false,
      rect: {},
      offset: {
        x: 0,
        y: 0,
      },
      initialX: 0,
      mousePos: {
        x: 0,
        y: 0,
      }
    };
  },

  mounted() {
    window.addEventListener('mouseup', this.handleMouseup);
    window.addEventListener('mousemove', this.handleMouseMove);
    this.rect = this.$el.getBoundingClientRect();
  },

  destroyed() {
    window.removeEventListener('mouseup', this.handleMouseup);
    window.removeEventListener('mousemove', this.handleMouseMove);
  },

  watch: {
    index() {
      if (!this.bodyGrabbed) {
        return;
      }

      const oldBox = {
        left: this.rect.left,
        top: this.rect.top,
      };

      this.$nextTick(() => {
        const newBox = this.$el.getBoundingClientRect();
        const delta = {
          x: newBox.left - oldBox.left,
          y: newBox.top - oldBox.top,
        };

        this.offset.x -= delta.x;
        this.offset.y -= delta.y;
      });

      this.$forceUpdate();
    }
  },

  computed: {
    style() {
      const { x, y } = this.offset;
      return {
        width: `calc(${(this.segment.length / this.totalLength) * 100}% - 2px)`,
        transform: this.bodyGrabbed ? `translate3D(${x}px, ${y}px, 0)` : 'translate3D(0,0,0)',
        'z-index': this.bodyGrabbed ? 1 : 0,
      };
    },
  },

  methods: {
    handleStartMousedown(e) {
      e.preventDefault();
      e.stopPropagation();
      this.$emit('start-grab');
      this.startGrabbed = true;
      this.initialX = e.screenX;
    },

    handleEndMousedown(e) {
      e.preventDefault();
      e.stopPropagation();
      this.$emit('end-grab');
      this.endGrabbed = true;
      this.initialX = e.screenX;
    },

    handleBodyMousedown(e) {
      this.bodyGrabbed = true;
      this.mousePos = {
        x: e.clientX,
        y: e.clientY,
      };
    },

    handleMouseMove(e) {
      if (!this.startGrabbed && !this.endGrabbed && !this.bodyGrabbed) {
        return;
      }

      const delta = {
        x: e.clientX - this.mousePos.x,
        y: e.clientY - this.mousePos.y,
      };

      this.mousePos = {
        x: e.clientX,
        y: e.clientY,
      };

      this.offset.x += delta.x;
      this.offset.y += delta.y;

      const resizeDiff = this.initialX - e.screenX;
      const pixelsPerSecond = this.getPixelsPerSecond();

      if (this.startGrabbed) {
        if (resizeDiff > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('lengthen-beginning');
        } else if ((resizeDiff * -1) > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('trim-beginning');
        }
      } else if (this.endGrabbed) {
        if (resizeDiff > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('trim-end');
        } else if ((resizeDiff * -1) > pixelsPerSecond) {
          this.initialX = e.screenX;
          this.$emit('lengthen-end');
        }
      } else if (this.bodyGrabbed) {
        this.rect = this.$el.getBoundingClientRect();

        this.$emit(
          'segment-moved',
          {
            offsetX: this.offset.x,
            left: this.rect.x,
            right: this.rect.x + this.rect.width,
          }
        );
      }
    },

    // We have to read from the DOM, so it's not great as a computed property
    getPixelsPerSecond() {
      const width = this.$el.clientWidth;
      const { length } = this.segment;
      return width / (length - 1);
    },

    clearOffset() {
      this.offset = {
        x: 0,
        y: 0,
      };
    },

    handleMouseup() {
      this.$emit('end-grab');
      this.clearOffset();
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
  color: white;
  height: 100%;
  width: 100%;
  border-radius: 8px;
  transition: background 0.1s ease-in-out;

  &.hidden {
    opacity: 0;
    pointer-events: none;
  }

  .handle {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 7px;
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

.grey {
  background: #848cab;

  &.timeline__segment--selected {
    background: #6b7394;
  }

  .handle {
    background: #6B7394;
  }
}

.teal {
  background: #21AD99;

  &.timeline__segment--selected,
  .handle {
    background: #008576;
  }
}

.turquoise {
  background: #26CFDB;

  &.timeline__segment--selected,
  .handle {
    background: #009CB8;
  }
}

.indigo {
  background: #8F97FF;

  &.timeline__segment--selected {
    background: #6361FA;
  }

  .handle {
    background: #6361FA;
  }
}

.video-title {
  display: block;
  font-size: 14px;
  font-weight: bold;
}

.video-range {
  font-size: 14px;
  color: #EBF0F7;
}

.video-range,
.video-title.video {
  width: 100%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  user-select: none;
}
</style>