<template>
  <div class="timeline-wrapper">
    <div class="timeline">
      <Segment
        v-for="(segment, index) in keyframes"
        :segment="segment"
        ref="segment"
        :key="segment.id"
        :selected="selected === index"
        :index="index"
        :name="videos[segment.video].name"
        :total-length="totalLength"
        @keydown.native="e => handleKeydown(e, index)"
        @select="() => $emit('select', index)"
        @lengthen-beginning="() => $emit('lengthen-beginning', index)"
        @trim-beginning="() => $emit('trim-beginning', index)"
        @lengthen-end="() => $emit('lengthen-end', index)"
        @trim-end="() => $emit('trim-end', index)"
        @start-grab="$emit('start-grab')"
        @end-grab="$emit('end-grab')"
        @segment-moved="(e) => handleSegmentMoved(e, index)"
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
            left: rect.x,
            right: rect.x + rect.width,
          };
        });
        console.log(this.positions);
      });
    }
  },

  methods: {
    handleSegmentMoved({ left, right }, index) {
      console.log('------');
      console.log(this.positions);
      console.log(left, right)

      // Check to see if have moved before the prior segment
      const segmentBefore = this.positions.find(pos => {
        return left > pos.left &&
               left < pos.right &&
               left < pos.left + 20;
      });

      if (segmentBefore) {
        this.$emit('move-left', index);
      }
    },

    handleKeydown(e, index) {
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
        this.$emit(eventName, index);
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
