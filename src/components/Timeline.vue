<template>
  <div class="timeline-wrapper">
    <div class="timeline">
      <Segment
        v-for="(segment, index) in keyframes"
        :segment="segment"
        :ref="segment.id"
        :key="segment.id"
        :selected="selected === segment.id"
        :index="index"
        :name="segment.name"
        :total-length="totalLength"
        :colour="segment.theme || 'grey'"
        :hidden="segment.name === '_'"
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
    <div class="add-video">
      <div class="overlay">+</div>
      <input v-if="type === 'video'" type="file" ref="fileInput" @change="handleInputChange" />
      <VyButton v-else @click="$emit('add-text-region')" />
    </div>
  </div>
</template>

<script>
import { VyButton } from '@vidyard/construction-yard';
import Segment from './Segment';

export default {
  name: "Timeline",

  components: {
    Segment,
    VyButton,
  },

  props: {
    keyframes: Array,
    time: Number,
    selected: String,
    totalLength: Number,
    type: String,
  },

  data() {
    return { positions: [] };
  },

  methods: {
    handleInputChange() {
      const file = this.$refs.fileInput.files[0];

      if (file) {
        this.$emit('add-video', file);
      }
    },

    handleSegmentMoved({ left, right, offsetX }, id) {
      const segmentBefore = this.keyframes.find(frame => {
        const box = this.$refs[frame.id][0].$el.getBoundingClientRect();
        return left > box.left &&
               left < box.left + Math.min(50, box.width / 2 - 10);
      });

      const segmentAfter = this.keyframes.find(frame => {
        const box = this.$refs[frame.id][0].$el.getBoundingClientRect();
        return right < box.right &&
               right > box.right - Math.min(50, box.width / 2 - 10);
      });

      if (segmentBefore && offsetX < 0) {
        this.$emit('move-left', id);
      } else if (segmentAfter && offsetX > 0) {
        this.$emit('move-right', id);
      }
    },

    handleKeydown(e, id) {
      if (this.selected === '') {
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
  padding: 20px 60px 20px 20px;
  position: relative;
}

.timeline {
  position: relative;
  margin: auto;
  width: 100%;
  max-width: 1200px;
  height: 70px;
}

.add-video {
  position: absolute;
  right: 10px;
  top: 45px;

  input,
  .vy-button {
    width: 50px !important;
    opacity: 0 !important;

    &:hover {
      cursor: pointer;
    }
  }

  &:hover {
    cursor: pointer;
  }
}

.overlay {
  top: -5px;
  left: 10px;
  position: absolute;
  width: 30px;
  height: 30px;
  text-align: center;
  font-size: 30px;
  line-height: 30px;
  border-radius: 50%;
  border: 1px solid #7518cc;
  color: #7518cc;
}

.current-time {
  position: absolute;
  top: -20px;
  height: calc(100% + 40px);
  width: 2px;
  background: #7518cc;
}


</style>
