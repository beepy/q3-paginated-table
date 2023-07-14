<script setup lang="ts">
import { getRecommendation, getNameMappings } from '../api';

const allProcedureTypes = ['SCS', 'BVN', 'SIJ'] as const;
type procedureType = (typeof allProcedureTypes)[number];

type recommendationType = {
  clinicId: string;
  patientAttributes: Record<string, string>;
  recDate: string;
  recommendedProcedures: procedureType[];
};

type nameMappingType = {
  header: Record<string, string[]>;
  column: Record<string, string>;
};

type labeledSelectValue = { title: string; value: string | number };

const recommendations = ref<recommendationType[]>([]);
const headers = ref<nameMappingType | undefined>(undefined);
const page = ref(1);
const pageSize = ref(3);
const totalPages = ref(0);
const totalRecords = ref(0);
const clinicValues: labeledSelectValue[] = [
  { title: 'Seattle Grace', value: 'seattle-grace' },
  { title: 'Sacred Heart', value: 'sacred-heart' },
];

const clinicId = ref<labeledSelectValue>(clinicValues[0].value);

onMounted(() => {
  // getRecommendation(clinicId.value, 3).then((d) => {
  //   recommendations.value = d.data;
  //   totalPages.value = d.totalPages;
  //   totalRecords.value = d.totalRecords;
  // });
  // refreshData();
});

watch(clinicId, () => {
  page.value = 1;
  headers.value = undefined;
  recommendations.value = [];
});
watch(pageSize, (newSize, oldSize) => {
  const firstItem = oldSize * (page.value - 1);
  page.value = Math.floor(firstItem / newSize) + 1;
});

watchEffect(() => {
  getRecommendation(clinicId.value, pageSize.value, page.value).then((d) => {
    recommendations.value = d.data;
    totalPages.value = d.totalPages;
    totalRecords.value = d.totalRecords;
  });
});

watchEffect(() => {
  getNameMappings(clinicId.value).then((d) => (headers.value = d));
});

const showingFirstItem = computed(() => (page.value - 1) * pageSize.value + 1);
const showingLastItem = computed(() =>
  Math.min(showingFirstItem.value + pageSize.value - 1, totalRecords.value)
);
</script>
<template>
  <div class="panel">
    <div style="display: flex; padding: 1rem; gap: 1rem">
      <v-select
        v-model="pageSize"
        :items="[1, 3]"
        variant="outlined"
        menu-icon="mdi-chevron-down"
        density="compact"
        style="flex-shrink: 1; flex-grow: 0"
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
    <v-table v-if="headers" style="">
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
          <td v-for="p in allProcedureTypes" :key="'rp-' + i + p">
            <template v-if="r.recommendedProcedures.find((n) => n.name === p)">
              <v-icon
                size="small"
                icon="mdi-check"
                style="color: #c52c9c"
              ></v-icon>
            </template>
          </td>
        </tr>
      </tbody>
    </v-table>
    <div style="display: flex; padding: 0.5rem 1rem; align-items: center">
      <div style="flex: 0 1 50%">
        <small style="color: #bcbbc1"
          >Showing {{ showingFirstItem }}
          <template v-if="showingFirstItem !== showingLastItem">
            to {{ showingLastItem }}
          </template>
          of {{ totalRecords }} entries
        </small>
      </div>
      <div style="flex: 1 1 50%; align-self: flex-end">
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
// light bg color #F1C8E5
// dark fg color #C52C9C
html,
body {
  background-color: #f5f4f7;
}
.v-table .v-table__wrapper > table > thead > tr:first-child > th {
  border-bottom: none;
  padding-top: 1em;
  --v-table-header-height: 0.5em;
}

.panel {
  margin: 2rem auto;
  border-radius: 6px;
  max-width: 80vw;
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.2);
  background-color: white;
}

.v-pagination__list {
  justify-content: flex-end;
}
.v-field.v-field--focused .v-field__outline {
  color: #c52c9c;
}

.v-pagination__item .v-btn__overlay {
  opacity: var(--v-border-opacity);
}

.v-pagination__item--is-active .v-btn__overlay {
  color: #c52c9c;
}
.panel .v-pagination {
  --v-border-opacity: 0.05;
  .v-pagination__prev,
  .v-pagination__next {
    .v-btn__overlay {
      opacity: var(--v-border-opacity);
    }
  }
}

.v-pagination {
  .v-pagination__prev,
  .v-pagination__next {
    .v-btn.v-btn--disabled {
      .v-btn__overlay {
        opacity: 0;
      }
    }
  }
}

.v-table__wrapper {
  border-top: thin solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-bottom: thin solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-radius: 0;
}
</style>
