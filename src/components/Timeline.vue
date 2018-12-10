<template>
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
      Video{{ segment.video }}
    </div>
    <div
      class="current-time"
      :style="{ left: `${ time / total * 100 }%` }"
    />
  </div>
</template>

<script>
export default {
  name: 'Timeline',

  props: {
    keyframes: Array,
    time: Number,
    selected: Number,
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

      switch (e.key) {
        case 'ArrowLeft':
          this.$emit('move-left', index);
          break;

        case 'ArrowRight':
          this.$emit('move-right', index);
          break;
      }
    }
  }
};
</script>

<style scoped>
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
  font-size: 14px;
  font-weight: bold;
  height: 100%;
  border-radius: 8px;
  transition: background 0.1s ease-in-out;
}

.timeline__segment:hover {
  cursor: pointer;
}

.timeline__segment:focus {
  outline: none;
  box-shadow: 0 0 0 3px #bfc4ff;
}

.timeline__segment--selected {
  background: #5B627D;
}
</style>
