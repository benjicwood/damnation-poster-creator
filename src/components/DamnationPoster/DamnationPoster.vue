<template>
  <div class="poster-container">
    <!-- Background selector buttons -->
    <div class="background-selector">
      <button
        :class="{ active: posterType === 'damnation' }"
        @click="posterType = 'damnation'"
      >
        DAMN
      </button>

      <button
        :class="{ active: posterType === 'salvation' }"
        @click="posterType = 'salvation'"
      >
        SALV
      </button>
    </div>
    <div
      class="poster-wrapper"
      ref="poster"
      @click.once="handleFirstInteraction"
    >
      <img
        class="poster-background"
        :src="backgroundSrc"
        alt="Festival Poster"
      />
      <BandGrid ref="bandGrid" :posterType="posterType" :alwaysHighlight="isMobile && posterEmpty" />
      <!-- Click to start overlay -->
      <div v-if="isMobile && !hasStartedEditing" class="click-start">
        CLICK A SECTION TO START
      </div>
    </div>
  </div>

  <!-- Fixed bottom bar -->
  <div class="button-row" :class="{ hidden: controlsHidden }">
    <div class="button-actions">
      <button
        class="download-btn"
        :disabled="isExporting"
        @click="downloadPoster"
      >
        Export Poster
      </button>

      <button
        v-if="isMobile"
        :disabled="isExporting"
        class="share-btn"
        @click="sharePoster"
      >
        Share Poster
      </button>

      <button
        v-else
        class="copy-btn"
        :disabled="isExporting"
        @click="copyPoster"
      >
        Copy to Clipboard
      </button>
    </div>

    <!-- <a
      href="/#/gallery"
      target="_blank"
      rel="noopener noreferrer"
      class="gallery-link"
    >
      Need inspiration? see more fan posters →
    </a> -->
  </div>

  <!-- Toggle button -->
  <button class="toggle-bar" @click="controlsHidden = !controlsHidden">
    {{ controlsHidden ? "▲" : "▼" }}
  </button>

  <!-- Toast -->
  <div v-if="toast.show" :class="['toast', toast.type]">
    {{ toast.message }}
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import { toPng, toBlob } from "html-to-image";

import BandGrid from "./BandGrid/BandGrid.vue";

import damnationBg from "../../assets/background/damnation27.jpg";
import salvationBg from "../../assets/background/salvation27.jpg";

const posterType = ref("damnation");

const poster = ref(null);
const bandGrid = ref(null);

const selectedYear = ref("2027");
const isMobile = ref(false);
const controlsHidden = ref(false);
const isExporting = ref(false);
const hasStartedEditing = ref(false);

const toast = ref({
  show: false,
  message: "",
  type: "success",
});

const posterEmpty = computed(() => {
  const grid = bandGrid.value;

  if (!grid) return true;

  return Object.values(grid.days).every((day) =>
    Object.values(day).every(
      (value) => !value?.bands || value.bands.length === 0,
    ),
  );
});

const backgroundSrc = computed(() => {
  return posterType.value === "salvation" ? salvationBg : damnationBg;
});
const showToast = (message, type = "success", duration = 2500) => {
  toast.value.show = true;
  toast.value.message = message;
  toast.value.type = type;

  setTimeout(() => {
    toast.value.show = false;
  }, duration);
};

const handleFirstInteraction = () => {
  hasStartedEditing.value = true;
};

const withCleanPosterRender = async (fn) => {
  const grid = bandGrid.value;

  if (!grid) {
    return fn();
  }

  await grid.runWithoutEditingUI(async () => {
    await fn();
  });
};

const isFirefox = () => {
  return navigator.userAgent.toLowerCase().includes("firefox");
};

const downloadPoster = async () => {
  if (isExporting.value) return;

  isExporting.value = true;

  try {
    await withCleanPosterRender(async () => {
      const node = poster.value;

      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
      const pixelRatio = isIOS ? 1 : 2;

      const blob = await toBlob(node, {
        backgroundColor: "#000",
        pixelRatio,
        skipFonts: isFirefox(),
      });

      const dataUrl = await toPng(node, {
        quality: 1,
        cacheBust: true,
        backgroundColor: "#000",
        pixelRatio,
        skipFonts: isFirefox(),
      });

      const link = document.createElement("a");

      link.download = `damnation-poster-${selectedYear.value}.png`;
      link.href = dataUrl;
      link.click();

      // if (!isIOS) {
      //   try {
      //     await uploadPosterToCloudinary(blob);
      //   } catch (error) {
      //     console.error(error);
      //   }
      // }

      showToast("Poster downloaded!");
    });
  } finally {
    isExporting.value = false;
  }
};

const sharePoster = async () => {
  if (isExporting.value) return;

  isExporting.value = true;

  try {
    await withCleanPosterRender(async () => {
      const node = poster.value;

      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
      const pixelRatio = isIOS ? 1 : 2;

      const blob = await toBlob(node, {
        backgroundColor: "#000",
        pixelRatio,
        skipFonts: isFirefox(),
      });

      const file = new File(
        [blob],
        `damnation-poster-${selectedYear.value}.png`,
        {
          type: "image/png",
        },
      );

      if (
        navigator.canShare &&
        navigator.canShare({
          files: [file],
        })
      ) {
        await navigator.share({
          files: [file],
          title: "Check out my festival poster!",
          text: `Made this Damnation Festival ${selectedYear.value} poster 🎶`,
        });

        showToast("Poster shared!");
      } else {
        const link = document.createElement("a");

        link.download = `damnation-poster-${selectedYear.value}.png`;
        link.href = URL.createObjectURL(blob);
        link.click();

        showToast(
          "Sharing not supported — poster downloaded instead.",
          "error",
        );
      }
    });
  } finally {
    isExporting.value = false;
  }
};

const copyPoster = async () => {
  if (isExporting.value) return;

  isExporting.value = true;

  try {
    await withCleanPosterRender(async () => {
      const node = poster.value;

      const blob = await toBlob(node, {
        backgroundColor: "#000",
        pixelRatio: 2,
        skipFonts: isFirefox(),
      });

      await navigator.clipboard.write([
        new ClipboardItem({
          "image/png": blob,
        }),
      ]);

      showToast("Poster copied to clipboard!");
    });
  } finally {
    isExporting.value = false;
  }
};

const uploadPosterToCloudinary = async (blob) => {
  const form = new FormData();

  form.append("file", blob);
  form.append("upload_preset", "damnation-posters");
  form.append("tags", "damnation-gallery");

  const response = await fetch(
    "https://api.cloudinary.com/v1_1/dhqkcdjcx/image/upload",
    {
      method: "POST",
      body: form,
    },
  );

  if (!response.ok) {
    const error = await response.text();
    throw new Error(error);
  }

  return response.json();
};

onMounted(() => {
  document.body.style.backgroundColor = "black";

  isMobile.value = /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);
});

onBeforeUnmount(() => {
  document.body.style.backgroundColor = "";
});
</script>

<style scoped lang="scss">
.poster-container {
  display: flex;
  // justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.poster-wrapper {
  position: relative;
  display: inline-block;
  margin: auto;
}

.poster-background {
  display: block;
  max-width: 98vw;
  max-height: 100vh;
  width: auto;
  height: auto;
}

/* --- Floating Control Bar --- */
.button-row {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
  display: flex;
  justify-content: center;
  gap: 0.75rem;
  background: #9b181a;
  padding: 0.75rem 1.25rem;
  border-radius: 12px 12px 0 0;
  backdrop-filter: blur(6px);
  transition:
    transform 0.3s ease,
    opacity 0.3s ease;

  &.hidden {
    transform: translate(-50%, 100%);
    opacity: 0;
    pointer-events: none;
  }
}

/* Toggle button */
.toggle-bar {
  font-family: sans-serif;
  border: 2px solid white;
  position: fixed;
  bottom: 4rem;
  // left: 50%;
  // transform: translateX(-50%);
  z-index: 101;
  background: #9b181a;
  color: white;
  border-radius: 9999px;
  padding: 0.4rem 0.6rem;
  cursor: pointer;
  font-size: 1.2rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  transition:
    background 0.2s ease,
    bottom 0.3s ease;

  &:hover {
    background: #111f18;
  }
}

/* Move toggle up slightly when bar is hidden */
.button-row.hidden + .toggle-bar {
  bottom: 0.5rem;
}

/* --- Buttons --- */
.download-btn,
.share-btn,
.copy-btn {
  background: #9b181a;
  border: 3px solid white;
  border-radius: 0;
  color: white;
  padding: 0.8rem 1.4rem;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.2s ease;

  &:hover {
    background: #e73370;
  }
}

.download-btn,
.share-btn,
.copy-btn {
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    pointer-events: none;
  }
}

.share-btn:hover {
  background: #2563eb;
}

.copy-btn:hover {
  background: #059669;
}

/* --- Toast --- */
.toast {
  position: fixed;
  bottom: 2rem;
  left: 50%;
  transform: translateX(-50%);
  padding: 0.8rem 1.2rem;
  color: white;
  font-weight: bold;
  z-index: 1000;
  opacity: 0.95;
  transition: opacity 0.3s ease;

  &.success {
    background-color: #10b981;
  }

  &.error {
    background-color: #ef4444;
  }
}

@media (max-width: 620px) {
  .poster-container {
    align-items: flex-start;
  }

  .poster-wrapper {
    margin: 4px auto;
  }

  .button-row {
    top: auto;
    right: auto;
    left: auto;
    transform: none;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    margin: 1.5rem auto 0 auto;
    background: transparent;
    box-shadow: none;
    padding: 0;
    bottom: 0;
    margin-bottom: 1rem;
  }
}

.click-start {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
  text-align: center;
  pointer-events: none;
  width: 90%;
}

.background-selector {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1000;
  display: flex;
  gap: 0.5rem; // space between buttons
  background: rgba(0, 0, 0, 0.5);
  padding: 0.5rem;
  border-radius: 6px;

  button {
    background: #333;
    color: white;
    border: 1px solid white;
    border-radius: 4px;
    padding: 0.5rem 1rem;
    cursor: pointer;
    font-family: sans-serif;
    font-weight: 600;
    transition: filter 0.2s;

    &:hover {
      filter: brightness(0.8); // darken on hover
    }

    &.active {
      background: #9b181a; // highlight active year
      color: white;
      filter: none;
      &:hover {
        filter: brightness(0.8); // darken on hover
      }
    }
  }

  // MOBILE: move to top-right
  @media (max-width: 768px) {
    left: auto;
    right: 10px;
  }
}

.button-row {
  flex-direction: column;
}

.button-actions {
  display: flex;
  gap: 0.75rem;
}

.gallery-link {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.85rem;
  text-decoration: none;
  text-align: center;
}

.gallery-link:hover {
  text-decoration: underline;
}
</style>
