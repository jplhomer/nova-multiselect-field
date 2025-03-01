<template>
  <default-field :field="field" :errors="errors">
    <template slot="field">
      <multiselect
        @input="handleChange"
        @open="() => repositionDropdown(true)"
        track-by="value"
        label="label"
        ref="multiselect"
        :value="selected"
        :options="options"
        :class="errorClasses"
        :placeholder="field.placeholder || field.name"
        :close-on-select="false"
        :clear-on-select="false"
        :multiple="true"
        :max="field.max || null"
        :optionsLimit="field.optionsLimit || 1000"
      ></multiselect>
    </template>
  </default-field>
</template>

<script>
import { FormField, HandlesValidationErrors } from 'laravel-nova';
import Multiselect from 'vue-multiselect';

export default {
  components: { Multiselect },

  mixins: [FormField, HandlesValidationErrors],

  props: ['resourceName', 'resourceId', 'field'],

  mounted() {
    window.addEventListener('scroll', this.repositionDropdown);
  },

  destroyed() {
    window.removeEventListener('scroll', this.repositionDropdown);
  },

  computed: {
    selected() {
      return this.value || [];
    },
    options() {
      return this.field.options || [];
    },
  },

  methods: {
    setInitialValue() {
      if (this.field.value) {
        let valuesArray;
        try {
          valuesArray = Array.isArray(this.field.value) ? this.field.value : JSON.parse(this.field.value);
          if (!Array.isArray(valuesArray)) return (this.value = []);
        } catch (e) {
          return (this.value = []);
        }

        this.value = valuesArray
          .map(val => this.field.options.find(opt => String(opt.value) === String(val)))
          .filter(Boolean);
      } else {
        this.value = [];
      }
    },

    fill(formData) {
      let value;
      if (this.value && this.value.length) {
        value = JSON.stringify(this.value.map(v => v.value));
      } else {
        value = this.field.nullable ? '' : JSON.stringify([]);
      }

      formData.append(this.field.attribute, value);
    },

    handleChange(value) {
      this.value = value;
      this.$nextTick(() => this.repositionDropdown());
    },

    repositionDropdown(onOpen = false) {
      const ms = this.$refs.multiselect;
      const el = ms.$el;

      const handlePositioning = () => {
        const { top, height, bottom } = el.getBoundingClientRect();
        if (onOpen) ms.$refs.list.scrollTop = 0;

        const fromBottom = (window.innerHeight || document.documentElement.clientHeight) - bottom;

        ms.$refs.list.style.position = 'fixed';
        ms.$refs.list.style.width = `${el.clientWidth}px`;

        if (fromBottom < 300) {
          ms.$refs.list.style.top = 'auto';
          ms.$refs.list.style.bottom = `${fromBottom + height}px`;
          ms.$refs.list.style['border-radius'] = '5px 5px 0 0';
        } else {
          ms.$refs.list.style.bottom = 'auto';
          ms.$refs.list.style.top = `${top + height}px`;
          ms.$refs.list.style['border-radius'] = '0 0 5px 5px';
        }
      };

      if (onOpen) this.$nextTick(handlePositioning);
      else handlePositioning();
    },
  },
};
</script>
