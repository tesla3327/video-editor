<template>
  <div class="sidebar">
    <h4>{{ title }}</h4>
    <!-- Handle editing text -->
    <template v-if="type === 'text' && selected">
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
      <label>
        <span>Font Size</span>
        <VyInputNumber
          :value="selected.fontSize"
          @input="$emit('change-font-size', $event)"
        />
      </label>
      <label>
        <span>Vertical Align</span>
        <VyMenuRadio
          labelled-by="stuff"
          :menu-items="positions"
          :selected="selectedPosition"
          @select="handleSelectPosition"
        />
      </label>
      <label>
        <span>Theme</span>
        <VyMenuRadio
          labelled-by="theme"
          :menu-items="themes"
          :selected="selectedTheme"
          @select="handleSelectTheme"
        />
      </label>
    </template>

    <!-- Handle editing video -->
    <template v-if="type === 'video' && selected">
      <VyInput
        label="Name"
        :value="selected.name"
        @input="$emit('change-name', $event)"
      />
      <label>
        <span>Theme</span>
        <VyMenuRadio
          labelled-by="theme"
          :menu-items="themes"
          :selected="selectedTheme"
          @select="handleSelectTheme"
        />
      </label>
    </template>
  </div>
</template>

<script>
import {
  VyInput,
  VyInputNumber,
  VyMenuRadio,
} from '@vidyard/construction-yard';

export default {
  name: 'Sidebar',

  props: {
    selected: Object,
    type: String,
  },

  components: {
    VyInput,
    VyInputNumber,
    VyMenuRadio,
  },

  data() {
    return {
      themes: [
        { text: 'grey' },
        { text: 'indigo' },
        { text: 'turquoise' },
        { text: 'teal' },
      ],
      positions: [
        { text: 'Top', value: 'flex-start' },
        { text: 'Center', value: 'center' },
        { text: 'Bottom', value: 'flex-end' },
      ]
    };
  },

  computed: {
    selectedTheme() {
      return this.themes.find(el => el.text === this.selected.theme);
    },

    selectedPosition() {
      return this.positions.find(el => el.value === this.selected.alignItems);
    },

    title() {
      return this.type === 'text'
        ? 'Edit Text Region'
        : 'Edit Video Region';
    },
  },

  methods: {
    handleSelectTheme(index) {
      this.$emit('change-theme', this.themes[index].text);
    },

    handleSelectPosition(index) {
      this.$emit('change-position', this.positions[index].value);
    },
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

  .vy-input-number,
  .menu-container {
    margin-bottom: 10px;
  }
}

.menu-container {
  margin-left: 0;
}

label span {
  font-size: 14px;
  font-weight: 300;
  display: block;
  padding-bottom: 7px;
}
</style>
