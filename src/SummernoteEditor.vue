<template>
  <div ref="summernoteRefElement"></div>
</template>

<script>
import $ from "jquery";
// You must import summernote js and css for yourself

// https://summernote.org/deep-dive/#callbacks
import events from "./events";

if (!window.SUMMERNOTE_DEFAULT_CONFIGS) {
  window.SUMMERNOTE_DEFAULT_CONFIGS = {};
}

export default {
  props: {
    modelValue: {
      default: null,
      required: true,
      event: "change",
      validator(value) {
        return (
          value === null || typeof value === "string" || value instanceof String
        );
      },
    },
    // https://summernote.org/deep-dive/
    config: {
      type: Object,
      default: () => window.SUMMERNOTE_DEFAULT_CONFIGS,
    },
  },
  data() {
    return {
      // jQuery DOM
      elem: null,
    };
  },
  mounted() {
    this.elem = $(this.$refs.summernoteRefElement);
    this.elem.summernote(this.config);
    $(this.elem).on("summernote.change", this.onChange);
    if (this.modelValue) {
      $(this.elem).summernote("code", this.modelValue);
    }
    this.registerEvents();
  },
  watch: {
    modelValue(newValue) {
      const currValue = $(this.elem).summernote("code");
      if (newValue != currValue) {
        $(this.elem).summernote("code", newValue);
      }
    },
  },
  methods: {
    onChange(event) {
      const value = $(this.elem).summernote("code");
      this.$emit("update:modelValue", value);
    },
    registerEvents() {
      for (var realName in events) {
        this.elem.on(`${realName}`, (...args) => {
          this.$emit(`${events[realName]}`, ...args);
        });
      }
    },
  },
  /**
   * Free up memory
   */
  beforeUnmount() {
    if (this.elem) {
      this.elem.summernote("destroy");
      this.elem = null;
    }
  },
};
</script>