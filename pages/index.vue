<script setup lang="ts">
import { getRecommendation, getNameMappings } from '../api';

const allProcedureTypes = ['BVN', 'SIJ', 'SCS'] as const;
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

const recommendations = ref<recommendationType[]>([]);
const headers = ref<nameMappingType | undefined>(undefined);
const clinicId = ref('seattle-grace');
const page = ref(1);
const totalPages = ref(0);
const totalRecords = ref(0);

onMounted(() => {
  getRecommendation(clinicId.value, 3).then((d) => {
    recommendations.value = d.data;
    totalPages.value = d.totalPages;
    totalRecords.value = d.totalRecords;
  });
  getNameMappings(clinicId.value).then((d) => (headers.value = d));
});
</script>
<template>
  <div>
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
            <small><strong>BVN</strong></small>
          </th>
          <th>
            <small><strong>SIJ</strong></small>
          </th>
          <th>
            <small><strong>SCS</strong></small>
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
              <span>X</span>
            </template>
          </td>
        </tr>
      </tbody>
    </v-table>
    <v-pagination v-model="page" :length="totalPages * 10"></v-pagination>
  </div>
</template>

<style lang="scss">
.v-table .v-table__wrapper > table > thead > tr:first-child > th {
  border-bottom: none;
  padding-top: 1em;
  --v-table-header-height: 0.5em;
}
</style>
