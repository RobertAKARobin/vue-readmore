<template>
  <div
    class="libReadMore"
    :class="{'libReadMore--expanded': isExpanded}"
  >
    <div
      class="libReadMore_content"
      :style="{ height: height }"
    >
      <div ref="content">
        <slot></slot>
      </div>
    </div>

    <div
      v-if="canBeExpanded"
      aria-hidden="true"
      class="libReadMore_toggle"
      :style="{'box-shadow': isExpanded ? 'none' : fade}"
      tabindex="0"
      @click.stop="toggleExpand"
    >
      <template v-if="isExpanded">
        {{ readLessText }}
      </template>
      <template v-else>
        {{ readMoreText }}
      </template>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'lib-read-more',
    props: {
      maxLines: {
        type: Number,
        default: 5,
      },
      readLessText: {
        type: String,
        default: 'Read less',
      },
      readMoreText: {
        type: String,
        default: 'Read more',
      },
      tolerancePx: {
        type: Number,
        default: 18,
      },
    },
    data() {
      return {
        contentHeight: false,
        isExpanded: false,
        isMounted: false,
        lineHeight: 0,
      };
    },
    computed: {
      canBeExpanded() {
        if (!this.isMounted) return false;
        return Boolean((this.contentHeight - this.tolerancePx) > this.maxHeight);
      },
      fade() {
        return `0 -${this.lineHeight / 2}px ${this.lineHeight / 2}px 0 #fff`;
      },
      height() {
        if (!this.canBeExpanded || this.isExpanded) {
          return 'auto';
        }
        return `${this.maxHeight}px`;
      },
    },
    watch: {
      maxLines() {
        this.setMaxHeight();
      },
      tolerancePx() {
        this.setMaxHeight();
      },
    },
    methods: {
      handleResize() {
        this.setContentHeight();
      },
      setContentHeight() {
        const content = this.$refs.content;
        this.contentHeight = content.getBoundingClientRect().height;
      },
      setMaxHeight() {
        const sampleElement = document.createElement('SPAN');
        sampleElement.style.whiteSpace = 'pre';
        sampleElement.style.position = 'absolute';
        sampleElement.style.visibility = 'hidden';
        sampleElement.innerHTML = Array.from(Array(this.maxLines).fill('A').join('\n'));
        this.$refs.content.appendChild(sampleElement);
        this.maxHeight = sampleElement.getBoundingClientRect().height;
        this.lineHeight = (this.maxHeight / this.maxLines);
        this.$refs.content.removeChild(sampleElement);
      },
      toggleExpand() {
        this.isExpanded = !(this.isExpanded);
        this.$emit('toggle', this.isExpanded);
      },
    },
    mounted() {
      this.isMounted = true;
      this.setMaxHeight();
      this.setContentHeight();
      window.addEventListener('resize', this.handleResize);
    },
    beforeDestroy() {
      window.removeEventListener('resize', this.handleResize);
    },
  };
</script>

<style lang="postcss" scoped>
.libReadMore_content {
  overflow: hidden;
}

.libReadMore_toggle {
  color: var(--blue600);
  cursor: pointer;
  display: block;
  position: relative;

  .libReadMore.libReadMore--expanded & {
    margin-top: 12px;
  }
}
</style>
