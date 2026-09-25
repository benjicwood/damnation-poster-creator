<template>
  <div v-if="posterType === 'damnation'" class="poster-content damnation-grid">
    <PosterRow
      class="headliner"
      :row="rows.headliner"
      placeholder="CLICK TO ADD HEADLINER(S)"
      @click="openEditor('headliner', 'Headliner')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('headliner')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      class="co-headliner"
      :row="rows.coHeadliner"
      placeholder="CLICK TO ADD CO-HEADLINER(S)"
      @click="openEditor('coHeadliner', 'Co-Headliner')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('coHeadliner')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      class="second-row"
      :row="rows.secondRow"
      placeholder="CLICK TO ADD MAIN SUPPORTS"
      @click="openEditor('secondRow', 'Main Support')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('secondRow')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      class="lower-lineup-one"
      :row="rows.lowerLineupOne"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('lowerLineupOne', 'Lower Lineup')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('lowerLineupOne')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      class="lower-lineup-two"
      :row="rows.lowerLineupTwo"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('lowerLineupTwo', 'Lower Lineup')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('lowerLineupTwo')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      class="lower-lineup-three"
      :row="rows.lowerLineupThree"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('lowerLineupThree', 'Lower Lineup')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('lowerLineupThree')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      class="lower-lineup-four"
      :row="rows.lowerLineupFour"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('lowerLineupFour', 'Lower Lineup')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('lowerLineupFour')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />
  </div>

  <div v-else class="poster-content salvation-grid">
    <PosterRow
      :row="salvationRows.rowOne"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('rowOne', 'Row One')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('rowOne')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      :row="salvationRows.rowTwo"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('rowTwo', 'Row Two')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('rowTwo')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />

    <PosterRow
      :row="salvationRows.rowThree"
      placeholder="CLICK TO ADD BANDS"
      @click="openEditor('rowThree', 'Row Three')"
      :alwaysHighlight="alwaysHighlight || isActiveRow('rowThree')"
      :showPlaceholderAlways="!posterStarted"
      :hideEditingUI="hideEditingUI"
    />
  </div>

  <RowEditorTray
    :title="editorTitle"
    f
    :row="isEditorVisible ? activeRow : null"
    slug="damnation"
    :rowKey="activeRowKey"
    :placement="trayPlacement"
    @add-band="addBandToRow"
    @remove-band="removeBandFromRow"
    @move-band="moveBandInRow"
    @set-size="setRowSize"
    @set-weight="setRowWeight"
    @set-align="setRowAlign"
    @set-divider="setRowDivider"
    @edit-band="editBandInRow"
    @close="closeEditor"
    @set-mode="setRowMode"
  />
</template>

<script setup>
import {
  reactive,
  ref,
  computed,
  nextTick,
  onMounted,
  onBeforeUnmount,
} from "vue";

import PosterRow from "./PosterRow.vue";
import RowEditorTray from "../../BandSelectModal/RowEditorTray.vue";
// import { trackBandSelected } from "../../../utils/analytics.js";

const props = defineProps({
  alwaysHighlight: {
    type: Boolean,
    default: false,
  },
  posterType: {
    type: String,
    default: "damnation",
  },
});

const makeRow = ({
  size = 5,
  weight = 500,
  maxBands = 5,
  minPx = 10,
  maxPx = 18,
  allowWrap = false,
  textAlign = "center",
  verticalAlign = "center",
  capSparseText = false,
  divider = "•",
  mode = "logos",
} = {}) => ({
  bands: [],
  size,
  weight,
  maxBands,
  minPx,
  maxPx,
  allowWrap,
  textAlign,
  verticalAlign,
  capSparseText,
  divider,
  mode,
});

const rows = reactive({
  headliner: makeRow({
    size: 6,
    weight: 900,
    maxBands: 2,
    minPx: 18,
    maxPx: 56,
    allowWrap: true,
    mode: "logos",
  }),

  coHeadliner: makeRow({
    size: 6,
    weight: 900,
    maxBands: 4,
    minPx: 18,
    maxPx: 50,
    allowWrap: true,
    mode: "logos",
  }),

  secondRow: makeRow({
    size: 6,
    weight: 500,
    maxBands: 6,
    minPx: 11,
    maxPx: 28,
    allowWrap: true,
    mode: "logos",
  }),

  lowerLineupOne: makeRow({
    size: 5,
    weight: 500,
    maxBands: 8,
    minPx: 8,
    maxPx: 18,
    allowWrap: true,
    verticalAlign: "top",
    capSparseText: true,
    mode: "logos",
  }),

  lowerLineupTwo: makeRow({
    size: 5,
    weight: 500,
    maxBands: 8,
    minPx: 8,
    maxPx: 18,
    allowWrap: true,
    verticalAlign: "top",
    capSparseText: true,
    mode: "logos",
  }),

  lowerLineupThree: makeRow({
    size: 5,
    weight: 500,
    maxBands: 8,
    minPx: 8,
    maxPx: 18,
    allowWrap: true,
    verticalAlign: "top",
    capSparseText: true,
    mode: "logos",
  }),

  lowerLineupFour: makeRow({
    size: 5,
    weight: 500,
    maxBands: 8,
    minPx: 8,
    maxPx: 18,
    allowWrap: true,
    verticalAlign: "top",
    capSparseText: true,
    mode: "logos",
  }),
});

const salvationRows = reactive({
  rowOne: makeRow({
    size: 6,
    weight: 900,
    maxBands: 2,
    minPx: 18,
    maxPx: 56,
    allowWrap: true,
    mode: "logos",
  }),

  rowTwo: makeRow({
    size: 5,
    weight: 700,
    maxBands: 2,
    minPx: 14,
    maxPx: 40,
    allowWrap: true,
    mode: "logos",
  }),

  rowThree: makeRow({
    size: 4,
    weight: 600,
    maxBands: 2,
    minPx: 12,
    maxPx: 32,
    allowWrap: true,
    mode: "logos",
  }),
});

const isEditorVisible = ref(false);
const activeRowKey = ref(null);
const editorTitle = ref("");
const isTrayMobile = ref(false);
const hideEditingUI = ref(false);

const activeRows = computed(() => {
  return props.posterType === "salvation" ? salvationRows : rows;
});

const activeRow = computed(() => {
  if (!activeRowKey.value) return null;

  return activeRows.value[activeRowKey.value] ?? null;
});

const posterStarted = computed(() =>
  Object.values(activeRows.value).some((row) => row.bands.length > 0),
);

// const trayPlacement = computed(() => {
//   return isTrayMobile.value ? "top" : "bottom";
// });

const trayPlacement = computed(() => {
  if (props.posterType === "salvation") {
    return activeRowKey.value === "rowThree" ? "top" : "bottom";
  }

  const topTrayRows = [
    "lowerLineupOne",
    "lowerLineupTwo",
    "lowerLineupThree",
    "lowerLineupFour",
  ];

  return topTrayRows.includes(activeRowKey.value) ? "top" : "bottom";
});

const updateTrayViewport = () => {
  isTrayMobile.value = window.innerWidth <= 700;
};

const openEditor = (rowKey, title) => {
  if (hideEditingUI.value) return;

  activeRowKey.value = rowKey;
  editorTitle.value = title;
  isEditorVisible.value = true;
};

const closeEditor = () => {
  isEditorVisible.value = false;
};

const isActiveRow = (rowKey) => {
  return (
    !hideEditingUI.value &&
    isEditorVisible.value &&
    activeRowKey.value === rowKey
  );
};

const normalizeBandName = (name) => {
  return (name || "").trim().toUpperCase();
};

const getAnalyticsRowLabel = (rowKey) => {
  const map = {
    headliner: "headliner",
    coHeadliner: "co_headliner",
    secondRow: "main_support",
    lowerLineupOne: "lower_lineup_1",
    lowerLineupTwo: "lower_lineup_2",
    lowerLineupThree: "lower_lineup_3",
    lowerLineupFour: "lower_lineup_4",
  };

  return map[rowKey] || rowKey;
};

// const trackBand = ({ id, name, source }) => {
//   trackBandSelected("poster_band_added", {
//     poster_day: "Damnation",
//     poster_row_key: activeRowKey.value,
//     poster_row_label: getAnalyticsRowLabel(activeRowKey.value),
//     band_id: id || name.toLowerCase().replace(/[^a-z0-9]+/g, ""),
//     band_name: name,
//     band_source: source || "custom",
//   });
// };

const addBandToRow = (band) => {
  const row = activeRow.value;

  if (!row) return;
  if (row.bands.length >= row.maxBands) return;

  const normalizedName = normalizeBandName(band.name);

  if (!normalizedName) return;

  const normalizedId = band.id || null;
  const normalizedSource = band.source || "custom";

  const alreadyExists = row.bands.some((existingBand) => {
    const existingName = normalizeBandName(existingBand.name);
    const existingId = existingBand.id || null;

    if (normalizedId && existingId) {
      return existingId === normalizedId;
    }

    return existingName === normalizedName;
  });

  if (alreadyExists) return;

  row.bands.push({
    id: normalizedId,
    name: normalizedName,
    source: normalizedSource,
  });

  // trackBand({
  //   id: normalizedId,
  //   name: normalizedName,
  //   source: normalizedSource,
  // });
};

const editBandInRow = ({ index, id, name, source }) => {
  const row = activeRow.value;

  if (!row || index < 0 || index >= row.bands.length) return;

  const normalizedName = normalizeBandName(name);

  row.bands[index] = {
    id: id || null,
    name: normalizedName,
    source: source || "custom",
  };

  // trackBand({
  //   id: id || null,
  //   name: normalizedName,
  //   source: source || "custom",
  // });
};

const removeBandFromRow = (index) => {
  const row = activeRow.value;

  if (!row || index < 0 || index >= row.bands.length) return;

  row.bands.splice(index, 1);
};

const moveBandInRow = ({ index, direction }) => {
  const row = activeRow.value;

  if (!row) return;

  const target = direction === "up" ? index - 1 : index + 1;

  if (target < 0 || target >= row.bands.length) return;

  const copy = [...row.bands];

  [copy[index], copy[target]] = [copy[target], copy[index]];

  row.bands = copy;
};

const setRowSize = (size) => {
  if (!activeRow.value) return;

  activeRow.value.size = Number(size) || 5;
};

const setRowWeight = (weight) => {
  if (!activeRow.value) return;

  activeRow.value.weight = Number(weight) || 500;
};

const setRowAlign = (align) => {
  if (!activeRow.value) return;

  activeRow.value.textAlign = align;
};

const setRowDivider = (divider) => {
  if (!activeRow.value) return;

  activeRow.value.divider = divider;
};

const setRowMode = (mode) => {
  if (!activeRow.value) return;

  activeRow.value.mode = mode === "logos" ? "logos" : "text";
};

const setPreviewMode = (isPreview) => {
  hideEditingUI.value = !!isPreview;

  if (isPreview) {
    closeEditor();
  }
};

const runWithoutEditingUI = async (fn) => {
  const previous = hideEditingUI.value;

  hideEditingUI.value = true;
  closeEditor();

  await nextTick();
  await new Promise((resolve) => requestAnimationFrame(resolve));

  try {
    return await fn();
  } finally {
    hideEditingUI.value = previous;
    await nextTick();
  }
};

onMounted(() => {
  updateTrayViewport();
  window.addEventListener("resize", updateTrayViewport);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", updateTrayViewport);
});

defineExpose({
  runWithoutEditingUI,
  setPreviewMode,
  rows,
  salvationRows,
});
</script>

<style scoped lang="scss">
@use "../../../assets/scss/styles.scss";

.poster-content {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;

  // padding-top: 17%;
  // padding-bottom: 8%;
  // padding-left: 7%;
  // padding-right: 4%;
}

.headliner {
  height: 20%;
}

.co-headliner {
  height: 18%;
}

.second-row {
  height: 12%;
}

.lower-lineup-one {
  height: 10%;
}

.lower-lineup-two {
  height: 10%;
}

.lower-lineup-three {
  height: 10%;
}
.lower-lineup-four {
  height: 10%;
}

.damnation-grid {
  padding-top: 17%;
  padding-bottom: 8%;
  padding-left: 2%;
  padding-right: 2%;
}

.salvation-grid {
  padding-top: 35%;
  padding-bottom: 15%;
  padding-left: 10%;
  padding-right: 10%;

  > * {
    height: 33.333%;
  }
}
</style>
