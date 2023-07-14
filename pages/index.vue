<script setup lang="ts">
import { getRecommendation, getNameMappings } from '../api';

const allProcedureTypes = ['SCS', 'BVN', 'SIJ'] as const;
type ProcedureType = (typeof allProcedureTypes)[number];

type RecommendationType = {
  clinicId: string;
  patientAttributes: Record<string, string | undefined>;
  recDate: string;
  recommendedProcedures: { name: ProcedureType }[];
};

type RecommendationResponse = {
  totalPages: number;
  totalRecords: number;
  data: RecommendationType[];
};

type NameMappingType = {
  header: Record<string, string[]>;
  column: Record<string, string>;
};

type LabeledSelectStringValue = { title: string; value: string };

const recommendations = ref<RecommendationType[]>([]);
const headers = ref<NameMappingType | undefined>(undefined);
const page = ref(1);
const pageSize = ref(3);
const totalPages = ref(0);
const totalRecords = ref(0);
const clinicValues: LabeledSelectStringValue[] = [
  { title: 'Sacred Heart', value: 'sacred-heart' },
  { title: 'Seattle Grace', value: 'seattle-grace' },
];

const clinicId = ref<string>(clinicValues[0].value as string);

// if the clinic ID changes, set the current page to 1
watch(clinicId, () => {
  page.value = 1;
  headers.value = undefined;
  recommendations.value = [];
});

// if the page size changes, set the current page so that the original first
// row is still visible
watch(pageSize, (newSize, oldSize) => {
  const firstItem = oldSize * (page.value - 1);
  page.value = Math.floor(firstItem / newSize) + 1;
});

// if the clinic, page, or page size changes, fetch the data
// (note: there are no cascading fetches when the clinic ID and page # change
// in the same pass 👍)
watchEffect(() => {
  getRecommendation(clinicId.value, pageSize.value, page.value).then((d) => {
    recommendations.value = d.data as RecommendationType[];
    totalPages.value = d.totalPages;
    totalRecords.value = d.totalRecords;
  });
});

// if the clinic ID changes, fetch the clinic's headers/name mappings
watchEffect(() => {
  getNameMappings(clinicId.value).then(
    (d) => (headers.value = d as NameMappingType)
  );
});

// 1-based index of the first and last item on the current page
const showingFirstItem = computed(() => (page.value - 1) * pageSize.value + 1);
const showingLastItem = computed(() =>
  Math.min(showingFirstItem.value + pageSize.value - 1, totalRecords.value)
);
</script>
<template>
  <div class="panel">
    <div class="table-controls">
      <v-select
        v-model="pageSize"
        :items="[1, 3]"
        variant="outlined"
        menu-icon="mdi-chevron-down"
        density="compact"
        class="select-page-size"
        hide-details
      />
      <v-select
        v-model="clinicId"
        label="Clinc"
        :items="clinicValues"
        variant="outlined"
        menu-icon="mdi-chevron-down"
        density="compact"
        hide-details
      ></v-select>
    </div>
    <v-table v-if="headers">
      <thead>
        <tr>
          <th
            v-for="k in Object.keys(headers.header)"
            :key="'header-' + k"
            :colspan="headers.header[k].length"
          >
            <small
              ><strong>{{ k }}</strong></small
            >
          </th>
          <th colspan="4">
            <small><strong>Recommendation</strong></small>
          </th>
        </tr>
        <tr>
          <th v-for="c in headers.column" :key="'hc-' + c">
            <small
              ><strong>{{ c }}</strong></small
            >
          </th>
          <th>
            <small><strong>Rec Date</strong></small>
          </th>
          <th>
            <small><strong>SCS</strong></small>
          </th>
          <th>
            <small><strong>BVN</strong></small>
          </th>
          <th>
            <small><strong>SIJ</strong></small>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(r, i) in recommendations" :key="r.clinicId + i">
          <td v-for="c in Object.keys(headers.column)" :key="'dc-' + c + i">
            {{ r.patientAttributes[c] }}
          </td>
          <td>{{ r.recDate }}</td>
          <td v-for="p in allProcedureTypes" :key="'rp-' + i + p" class="check">
            <template v-if="r.recommendedProcedures.find((n) => n.name === p)">
              <v-icon size="small" icon="mdi-check"></v-icon>
            </template>
          </td>
        </tr>
      </tbody>
    </v-table>
    <div class="table-footer">
      <div class="details">
        <small
          >Showing {{ showingFirstItem }}
          <template v-if="showingFirstItem !== showingLastItem">
            to {{ showingLastItem }}
          </template>
          of {{ totalRecords }} entries
        </small>
      </div>
      <div class="pagination">
        <v-pagination
          v-model="page"
          :length="totalPages"
          density="compact"
          variant="flat"
          :total-visible="5"
        />
      </div>
    </div>
  </div>
</template>

<style lang="scss">
$brand-fg: #c52c9c;
$light-gray: #bcbbc1;
$disabled-pagination-button-opacity: 0.07;

html,
body {
  background-color: #f5f4f7;
}

.panel {
  margin: 2rem auto;
  border-radius: 6px;
  max-width: min(calc(100vw - 2rem), 960px);
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.2);
  background-color: white;
}

.table-controls {
  display: flex;
  padding: 1rem;
  gap: 1rem;
  .select-page-size {
    flex: 0 1 auto;
  }
}

table td.check {
  color: $brand-fg;
}

.table-footer {
  display: flex;
  padding: 0.5rem 1rem;
  align-items: center;
  .details {
    flex: 0 1 50%;
    color: $light-gray;
  }
  .pagination {
    flex: 1 1 40%;
    align-self: flex-end;
  }
}

///////////////////////////
// Vuetify customization //
///////////////////////////

// outline color on focused controls
.v-field.v-field--focused .v-field__outline {
  color: $brand-fg;
}

// select item - brand color highlight, inset rounded corners
.v-list-item--density-default.v-list-item--one-line {
  min-height: 30px;
  margin-bottom: 0.25rem;
}

.v-list-item--variant-text .v-list-item__overlay {
  max-width: calc(100% - 1rem);
  left: 0.5rem;
  color: $brand-fg;
  border-radius: 4px;
}

// table - shorter first row of table header
.v-table .v-table__wrapper > table > thead > tr:first-child > th {
  border-bottom: none;
  padding-top: 1em;
  --v-table-header-height: 0.5em;
}

// table - top and bottom border; no border radius
.v-table__wrapper {
  border-top: thin solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-bottom: thin solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-radius: 0;
}

// pagination - colors, backgrounds, alignment
.v-pagination {
  --v-border-opacity: 0.05;
  .v-pagination__prev,
  .v-pagination__next {
    .v-btn__overlay {
      opacity: var(--v-border-opacity);
    }
    .v-btn.v-btn--disabled {
      .v-btn__overlay {
        opacity: $disabled-pagination-button-opacity;
      }
    }
  }
  .v-btn--disabled.v-btn--variant-flat .v-btn__overlay {
    opacity: $disabled-pagination-button-opacity;
  }
}

.v-pagination__list {
  justify-content: flex-end;
}

.v-pagination__item .v-btn__overlay {
  opacity: var(--v-border-opacity);
}

.v-pagination__item--is-active {
  .v-btn__overlay,
  .v-btn__content {
    color: $brand-fg;
  }
}
</style>
