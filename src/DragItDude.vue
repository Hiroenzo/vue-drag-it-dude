<template>
  <div class="drag-it-dude" @mousedown.stop="hang" @mouseup.stop="drop">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "drag-it-dude",
  props: {
    width: {
      type: Number,
      default: 0,
    },
    height: {
      type: Number,
      default: 0,
    },
    parentWidth: {
      type: Number,
      default: 0,
    },
    parentHeight: {
      type: Number,
      default: 0,
    },
  },
  watch: {
    width(newWidth, oldWidth) {
      if (newWidth < oldWidth) return;
      if (this.left === 0) return;

      this.parent.width = this.parentWidth || this.elem.parentNode.offsetWidth;
      this.parent.height =
        this.parentHeight || this.elem.parentNode.offsetHeight;

      if (newWidth > this.parent.width - this.left) {
        const newLeft = this.parent.width - newWidth;
        this.left = newLeft < 0 ? 0 : newLeft;
        this.elem.style.left = `${this.left}px`;
      }
    },
    height(newHeight, oldHeight) {
      if (newHeight < oldHeight) return;
      if (this.top === 0) return;

      this.parent.width = this.parentWidth || this.elem.parentNode.offsetWidth;
      this.parent.height =
        this.parentHeight || this.elem.parentNode.offsetHeight;

      if (newHeight > this.parent.height - this.top) {
        const newTop = this.parent.height - this.height;
        this.top = newTop;
        this.elem.style.top = `${this.top}px`;
      }
    },
  },
  data: () => ({
    shiftY: null,
    shiftX: null,
    left: 0,
    top: 0,
    elem: null,
    parent: {
      width: 0,
      height: 0,
    },
  }),
  methods: {
    move(e) {
      this.$emit("dragging");
      e.preventDefault();
      if (!e.pageX) {
        document.body.style.overflow = "hidden";
      }
      const x = e.pageX || e.changedTouches[0].pageX;
      const y = e.pageY || e.changedTouches[0].pageY;
      let newLeft = x - this.shiftX;
      let newTop = y - this.shiftY;
      const newRight = newLeft + this.elem.offsetWidth;
      const newBottom = newTop + this.elem.offsetHeight;
      if (newLeft < 0) {
        newLeft = 0;
      } else if (newRight > this.parent.width) {
        newLeft = this.parent.width - this.elem.offsetWidth;
      } else {
        newLeft = x - this.shiftX;
      }
      if (newTop < 0) {
        newTop = 0;
      } else if (newBottom > this.parent.height) {
        newTop = this.parent.height - this.elem.offsetHeight;
      } else {
        newTop = y - this.shiftY;
      }
      this.elem.style.left = `${newLeft}px`;
      this.left = newLeft;
      this.elem.style.top = `${newTop}px`;
      this.top = newTop;
    },
    hang(e) {
      this.$emit("activated");
      this.parent.width = this.parentWidth || this.elem.parentNode.offsetWidth;
      this.parent.height =
        this.parentHeight || this.elem.parentNode.offsetHeight;

      const x = e.pageX || e.changedTouches[0].pageX;
      const y = e.pageY || e.changedTouches[0].pageY;
      this.shiftX = x - this.elem.offsetLeft;
      this.shiftY = y - this.elem.offsetTop;
      if (e.pageX) {
        this.elem.addEventListener("mousemove", this.move);
        this.elem.addEventListener("mouseleave", this.drop);
      }
    },
    drop() {
      this.$emit("dropped");
      document.body.style.overflow = null;
      this.elem.removeEventListener("mousemove", this.move, false);
      this.elem.onmouseup = null;
    },
  },
  mounted() {
    this.elem = this.$el;
  },
};
</script>

<style>
.drag-it-dude {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}
</style>
