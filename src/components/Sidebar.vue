<template>
  <div class="sidebar">
    <h4>{{ title }}</h4>
    <!-- Handle editing text -->
    <template v-if="type === 'text'">
      <VyInput
        label="Text"
        :value="selected.text"
        @input="$emit('change-text', $event)"
      />
      <VyInput
        label="Colour"
        :value="selected.colour"
        @input="$emit('change-colour', $event)"
      />
      <VyInput
        label="Font Size"
        :value="selected.fontSize"
        @input="$emit('change-font-size', $event)"
      />
    </template>
  </div>
</template>

<script>
import {
  VyInput,
} from '@vidyard/construction-yard';

export default {
  name: 'Sidebar',

  props: {
    selected: Object,
    type: String,
  },

  components: {
    VyInput,
  },

  computed: {
    title() {
      return this.type === 'text'
        ? 'Edit Text Region'
        : 'Edit Video Region';
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../../node_modules/@vidyard/construction-yard/dist/system/system.css';
@import '../../node_modules/@vidyard/construction-yard/dist/system/system.utils.scss';

.sidebar {
  display: flex;
  flex-flow: column;
  width: 300px;
  height: 100%;
  border-right: 1px solid $c-grey-200;
  padding: $space-small;

  h4 {
    margin: 0;
    padding: 0 0 $space-small 0;
  }

  .vy-input {
    padding-bottom: $space-small;
  }
}
</style>
