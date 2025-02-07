<template>
  <div>
    <div ref="point"
         :class="{'avue-affix':affix}"
         :style="styles">
      <slot></slot>
    </div>
    <div v-show="slot"
         :style="slotStyle"></div>
  </div>
</template>
<script>
import create from "core/create";
export default create({
  name: "affix",
  props: {
    target: String,
    offsetTop: {
      type: Number,
      default: 0
    },
    offsetBottom: {
      type: Number
    }
  },
  data () {
    return {
      container: null,
      affix: false,
      styles: {},
      slot: false,
      slotStyle: {}
    };
  },
  computed: {
    offsetType () {
      let type = "top";
      if (this.offsetBottom >= 0) {
        type = "bottom";
      }
      return type;
    }
  },
  mounted () {
    if (this.target) {
      this.container = document.querySelector(this.target);
    } else {
      this.container = document
    }
    this.container.addEventListener("scroll", this.handleScroll, false);
    this.container.addEventListener("resize", this.handleScroll, false);
  },
  methods: {
    getScroll (target, top) {
      const prop = top ? "pageYOffset" : "pageXOffset";
      const method = top ? "scrollTop" : "scrollLeft";

      let ret = target[prop];
      if (typeof ret !== "number") {
        ret = document.documentElement[method];
      }

      return ret;
    },

    getOffset (element) {
      const rect = element.getBoundingClientRect();

      const scrollTop = this.getScroll(this.container, true);
      const scrollLeft = this.getScroll(this.container);

      const docEl = document.body;
      const clientTop = docEl.clientTop || 0;
      const clientLeft = docEl.clientLeft || 0;

      return {
        top: rect.top + scrollTop - clientTop,
        left: rect.left + scrollLeft - clientLeft
      };
    },
    handleScroll () {
      const affix = this.affix;
      const scrollTop = this.getScroll(this.container, true);
      const elOffset = this.getOffset(this.$el);
      const windowHeight = this.container.innerHeight;
      const elHeight = this.$el.getElementsByTagName('div')[0].offsetHeight;

      if ((elOffset.top - this.offsetTop) < scrollTop && this.offsetType == 'top' && !affix) {
        this.affix = true;
        this.slotStyle = {
          width: this.$refs.point.clientWidth + 'px',
          height: this.$refs.point.clientHeight + 'px'
        };
        this.slot = true;
        this.styles = {
          top: `${this.offsetTop}px`,
          left: `${elOffset.left}px`,
          width: `${this.$el.offsetWidth}px`
        };

        this.$emit('on-change', true);
      } else if ((elOffset.top - this.offsetTop) > scrollTop && this.offsetType == 'top' && affix) {
        this.slot = false;
        this.slotStyle = {};
        this.affix = false;
        this.styles = null;

        this.$emit('on-change', false);
      }

      if ((elOffset.top + this.offsetBottom + elHeight) > (scrollTop + windowHeight) && this.offsetType == 'bottom' && !affix) {
        this.affix = true;
        this.styles = {
          bottom: `${this.offsetBottom}px`,
          left: `${elOffset.left}px`,
          width: `${this.$el.offsetWidth}px`
        };

        this.$emit('on-change', true);
      } else if ((elOffset.top + this.offsetBottom + elHeight) < (scrollTop + windowHeight) && this.offsetType == 'bottom' && affix) {
        this.affix = false;
        this.styles = null;

        this.$emit('on-change', false);
      }
    }
  },
  beforeDestroy () {
    this.container.removeEventListener("scroll", this.handleScroll, false);
    this.container.removeEventListener("resize", this.handleScroll, false);
  }
});
</script>