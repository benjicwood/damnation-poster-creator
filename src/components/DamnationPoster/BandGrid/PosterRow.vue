<template>
  <div
    ref="rowEl"
    :class="[
      'poster-row',
      rowClass,
      `align-${row?.textAlign || 'center'}`,
      `valign-${row?.verticalAlign || 'center'}`,
      {
        'always-highlight': alwaysHighlight && !hideEditingUI,
        'editing-ui-hidden': hideEditingUI,
      },
    ]"
    @click="$emit('click')"
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <div v-if="showLogoLayout" class="poster-logo-row">
      <div
        v-for="(band, index) in logoBands"
        :key="`${band.id || band.name}-${index}`"
        class="poster-logo-item"
        :style="band.logo ? { transform: `scale(${logoScale})` } : {}"
      >
        <img
          v-if="band.logo"
          :src="band.logo"
          :alt="band.name"
          class="poster-row-logo"
        />

        <span
          v-else
          class="poster-logo-fallback"
          :style="{ fontSize: `${getLogoFallbackSize(band.name)}px` }"
        >
          {{ band.name }}
        </span>
      </div>
    </div>

    <span v-else-if="displayText" class="poster-row-text" :style="textStyle">
      {{ displayText }}
    </span>

    <span v-else-if="shouldShowPlaceholder" class="poster-row-placeholder">
      {{ placeholder }}
    </span>
  </div>
</template>

<script>
import { bands as artistBands } from "@benjicwood/artist-assets";

export default {
  name: "PosterRow",
  emits: ["click"],
  props: {
    row: { type: Object, required: true },
    placeholder: { type: String, default: "CLICK TO ADD BANDS" },
    alwaysHighlight: { type: Boolean, default: false },
    showPlaceholderAlways: { type: Boolean, default: false },
    hideEditingUI: { type: Boolean, default: false },
  },

  data() {
    return {
      resolvedPx: 18,
      resolvedLetterSpacing: "0.04em",
      resizeObserver: null,
      isHovered: false,
    };
  },

  computed: {
    isLogoMode() {
      return this.row?.mode === "logos";
    },

    logoBands() {
      if (!this.isLogoMode) return [];

      return (this.row?.bands || []).map((band) => {
        const assetBand = band?.id
          ? artistBands.find((b) => b.id === band.id)
          : null;

        return {
          ...band,
          logo: assetBand?.logo || null,
          name: (band?.name || "").trim().toUpperCase(),
        };
      });
    },

    displayText() {
      const divider = this.row?.divider ?? "•";
      const joiner =
        divider === " " ? "\u00A0\u00A0\u00A0\u00A0" : ` ${divider} `;

      return (this.row?.bands || [])
        .map((band) => (band?.name || "").trim().toUpperCase())
        .filter(Boolean)
        .join(joiner);
    },

    shouldShowPlaceholder() {
      if (this.hideEditingUI) return false;
      if (this.row?.bands?.length) return false;
      if (this.displayText) return false;
      if (this.showPlaceholderAlways) return true;
      return this.isHovered || this.alwaysHighlight;
    },

    basePx() {
      const size = Number(this.row?.size) || 5;
      const min = Number(this.row?.minPx) || 10;
      const max = Number(this.row?.maxPx) || 18;

      const t = (size - 1) / 9;
      return Math.round(min + t * (max - min));
    },

    minPx() {
      return this.row?.minPx || 10;
    },

    baseLetterSpacing() {
      return 0;
    },

    rowClass() {
      if (this.row?.size <= 3) return "is-small-row";
      if (this.row?.size >= 8) return "is-headliner-row";
      return "";
    },

    textStyle() {
      return {
        fontSize: `${this.resolvedPx}px`,
        fontWeight: "400",
        fontStyle: "italic",
        whiteSpace: this.row?.allowWrap ? "normal" : "nowrap",
        lineHeight: this.row?.allowWrap ? "1.05" : "1",
        letterSpacing: this.resolvedLetterSpacing,
        textAlign: this.row?.textAlign || "center",
        margin: "auto",
      };
    },
    showLogoLayout() {
      return (
        this.isLogoMode &&
        this.logoBands.length &&
        this.logoBands.some((band) => band.logo)
      );
    },
    logoScale() {
      const size = Number(this.row?.size || 5);

      const scaleMap = {
        1: 0.4,
        2: 0.5,
        3: 0.6,
        4: 0.75,
        5: 0.9,
        6: 1,
        7: 1.1,
        8: 1.2,
        9: 1.3,
        10: 1.4,
      };

      return scaleMap[size] || 6;
    },
  },

  mounted() {
    this.$nextTick(this.fitRowText);
    window.addEventListener("resize", this.fitRowText);

    if ("ResizeObserver" in window && this.$refs.rowEl instanceof Element) {
      this.resizeObserver = new ResizeObserver(() => this.fitRowText());
      this.resizeObserver.observe(this.$refs.rowEl);
    }
  },

  beforeUnmount() {
    window.removeEventListener("resize", this.fitRowText);
    if (this.resizeObserver) this.resizeObserver.disconnect();
  },

  watch: {
    row: {
      deep: true,
      handler() {
        this.$nextTick(this.fitRowText);
      },
    },
  },

  methods: {
    fitRowText() {
      const container = this.$refs.rowEl;

      if (!container || !this.displayText || this.showLogoLayout) {
        this.resolvedPx = this.basePx;
        this.resolvedLetterSpacing = `${this.baseLetterSpacing}em`;
        return;
      }

      const maxWidth = Math.max(container.clientWidth - 10, 0);
      const maxHeight = Math.max(container.clientHeight - 8, 0);

      if (!maxWidth || !maxHeight) {
        this.resolvedPx = this.basePx;
        this.resolvedLetterSpacing = `${this.baseLetterSpacing}em`;
        return;
      }

      let fittedPx = this.minPx;

      for (let px = this.row?.maxPx || 120; px >= this.minPx; px--) {
        if (
          this.textFits({
            text: this.displayText,
            fontSize: px,
            weight: this.row.weight,
            letterSpacing: this.baseLetterSpacing,
            maxWidth,
            maxHeight,
            allowWrap: !!this.row.allowWrap,
          })
        ) {
          fittedPx = px;
          break;
        }
      }

      const scaleMap = {
        1: 0.55,
        2: 0.65,
        3: 0.75,
        4: 0.85,
        5: 0.95,
        6: 1.0,
        7: 1.05,
        8: 1.1,
        9: 1.15,
        10: 1.2,
      };

      const size = Number(this.row?.size) || 5;
      let targetPx = Math.round(fittedPx * (scaleMap[size] || 1));

      const isMobile = window.innerWidth <= 700;

      for (let px = targetPx; px >= this.minPx; px--) {
        if (
          this.textFits({
            text: this.displayText,
            fontSize: px,
            weight: this.row.weight,
            letterSpacing: this.baseLetterSpacing,
            maxWidth,
            maxHeight,
            allowWrap: !!this.row.allowWrap,
          })
        ) {
          this.resolvedPx = px;
          this.resolvedLetterSpacing = `${this.baseLetterSpacing}em`;
          return;
        }
      }

      this.resolvedPx = this.minPx;
      this.resolvedLetterSpacing = `${this.baseLetterSpacing}em`;
    },

    getLogoFallbackSize(name) {
      const size = Number(this.row?.size || 5);
      const bandCount = this.logoBands.length || 1;

      const sizeMap = {
        1: 22,
        2: 25,
        3: 28,
        4: 31,
        5: 35,
        6: 40,
        7: 46,
        8: 52,
        9: 55,
        10: 58,
      };

      const bandScale = {
        1: 1,
        2: 1,
        3: 0.9,
        4: 0.8,
        5: 0.7,
        6: 0.6,
        7: 0.5,
        8: 0.45,
      };

      return sizeMap[size] * (bandScale[bandCount] || 0.5);
    },

    textFits({
      text,
      fontSize,
      weight,
      letterSpacing,
      maxWidth,
      maxHeight,
      allowWrap,
    }) {
      const measure = document.createElement("span");
      measure.textContent = text;
      measure.style.position = "absolute";
      measure.style.visibility = "hidden";
      measure.style.pointerEvents = "none";
      measure.style.left = "-99999px";
      measure.style.top = "-99999px";
      measure.style.display = "block";
      measure.style.fontFamily = '"GrimoireOfDeath", serif';
      measure.style.fontSize = `${fontSize}px`;
      measure.style.fontWeight = "400";
      measure.style.fontStyle = "italic";
      measure.style.letterSpacing = `${letterSpacing}em`;
      measure.style.textTransform = "uppercase";
      measure.style.lineHeight = allowWrap ? "1.05" : "1";
      measure.style.whiteSpace = allowWrap ? "normal" : "nowrap";
      measure.style.width = allowWrap ? `${maxWidth}px` : "auto";
      measure.style.maxWidth = `${maxWidth}px`;

      document.body.appendChild(measure);

      const fits =
        measure.offsetWidth <= maxWidth + 1 &&
        measure.offsetHeight <= maxHeight + 1;

      document.body.removeChild(measure);
      return fits;
    },
  },
};
</script>

<style scoped lang="scss">
@font-face {
  font-family: "GrimoireOfDeath";
  src: url("/fonts/GrimoireOfDeath-Italic.ttf") format("truetype");
  font-weight: 400;
  font-style: italic;
  font-display: swap;
}

.poster-row {
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  box-sizing: border-box;
  min-width: 0;
  min-height: 0;
  overflow: hidden;
  padding: 0.25rem 0.35rem;
  border: 1px solid transparent;
}

.poster-row:hover:not(.editing-ui-hidden) {
  border-color: #c67d0e;
}

// .poster-row-text {
//   display: block;
//   width: 100%;
//   color: white;
//   font-family: "NeueHaasUnica", sans-serif;
//   text-transform: uppercase;
//   font-synthesis: none;
// }

.poster-row-text {
  display: block;
  width: 100%;
  color: white;
  font-family: "GrimoireOfDeath", serif;
  font-style: italic;
  font-weight: 400;
  text-transform: uppercase;
  font-synthesis: none;
}

.poster-row-placeholder {
  color: rgba(255, 255, 255, 0.45);
  font-family: sans-serif;
  font-size: 1rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  margin: auto;
}

.poster-logo-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 3%;
  width: 100%;
  height: 100%;
  min-width: 0;
}

.poster-logo-item {
  flex: 1;
  min-width: 0;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.poster-row-logo {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  display: block;
}

// .poster-logo-fallback {
//   color: white;
//   font-family: "NeueHaasUnica", sans-serif;

//   text-transform: uppercase;
//   // font-size: clamp(10px, 1vw, 20px);
//   line-height: 1;
//   text-align: center;
// }
.poster-logo-fallback {
  color: white;
  font-family: "GrimoireOfDeath", serif;
  font-style: italic;
  font-weight: 400;
  text-transform: uppercase;
  line-height: 1;
  text-align: center;
}

.align-left {
  justify-content: flex-start;
}

.align-center {
  justify-content: center;
}

.align-right {
  justify-content: flex-end;
}

.valign-top {
  align-items: flex-start;
  padding-top: 0.4rem;
}

.valign-center {
  align-items: center;
}

.valign-bottom {
  align-items: flex-end;
}

.is-small-row .poster-row-text {
  text-rendering: geometricPrecision;
}

.is-headliner-row .poster-row-text {
  text-wrap: balance;
}

.always-highlight {
  border-color: #c67d0e !important;
}

.editing-ui-hidden {
  border-color: transparent !important;
}
</style>
