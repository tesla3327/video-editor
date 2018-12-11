<template>
  <div class="timeline-wrapper">
    <div class="timeline">
      <Segment
        v-for="(segment, index) in keyframes"
        :segment="segment"
        ref="segment"
        :key="segment.id"
        :selected="selected === segment.id"
        :index="index"
        :name="videos[segment.video].name"
        :total-length="totalLength"
        @keydown.native="e => handleKeydown(e, segment.id)"
        @select="() => $emit('select', segment.id)"
        @lengthen-beginning="() => $emit('lengthen-beginning', segment.id)"
        @trim-beginning="() => $emit('trim-beginning', segment.id)"
        @lengthen-end="() => $emit('lengthen-end', segment.id)"
        @trim-end="() => $emit('trim-end', segment.id)"
        @start-grab="$emit('start-grab')"
        @end-grab="$emit('end-grab')"
        @segment-moved="(e) => handleSegmentMoved(e, segment.id)"
        tabindex="0"
      />
      <div
        class="current-time"
        :style="{ left: `${(time / totalLength) * 100}%` }"
      />
    </div>
  </div>
</template>

<script>
import Segment from './Segment';

export default {
  name: "Timeline",

  components: {
    Segment,
  },

  props: {
    keyframes: Array,
    time: Number,
    selected: Number,
    videos: Array,
    totalLength: Number
  },

  data() {
    return { positions: [] };
  },

  watch: {
    keyframes() {
      this.$nextTick(() => {
        this.positions = this.keyframes.map((keyframe, index) => {
          if (!this.$refs.segment) {
            return [];
          }

          const rect = this.$refs.segment[index].$el.getBoundingClientRect();
          return {
            id: keyframe.id,
            left: rect.x,
            right: rect.x + rect.width,
          };
        });
      });
    }
  },

  methods: {
    handleSegmentMoved({ left, right }, id) {
      // Check to see if have moved before the prior segment
      const segmentBefore = this.positions
        .filter(pos => id !== pos.id)
        .find(pos => {
          return left > pos.left &&
                left < pos.right - 50
        });

      if (segmentBefore) {
        this.$emit('move-left', id);
      }
    },

    handleKeydown(e, id) {
      if (this.selected === -1) {
        return;
      }

      let eventName;

      if (e.key === "ArrowLeft") {
        if (e.shiftKey) {
          eventName = "lengthen-beginning";
        } else if (e.metaKey) {
          eventName = "trim-end";
        } else {
          eventName = "move-left";
        }
      } else if (e.key === "ArrowRight") {
        if (e.shiftKey) {
          eventName = "trim-beginning";
        } else if (e.metaKey) {
          eventName = "lengthen-end";
        } else {
          eventName = "move-right";
        }
      } else if (e.code === "KeyS") {
        eventName = "split";
      } else if (e.code === "KeyD" && e.metaKey) {
        eventName = "duplicate";
      } else if (e.code === "Delete" || e.code === "Backspace") {
        eventName = "delete";
      }

      if (eventName) {
        e.preventDefault();
        e.stopPropagation();
        this.$emit(eventName, id);
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.timeline-wrapper {
  background: #c8cee3;
  padding: 20px;
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
  top: -20px;
  height: calc(100% + 40px);
  width: 2px;
  background: #7518cc;
  box-shadow: 0 10px 30px 0px rgba(0, 0, 0, 0.8);
}


</style>
