<template>
  <div>
    <v-toolbar light>
      <v-toolbar-title> Manage Observations </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn color="primary" to="/main/observation/observations/create"
        >Create Observation</v-btn
      >
    </v-toolbar>
    <v-data-table :headers="headers" :items="observations">
      <!-- eslint-disable-next-line vue/valid-v-slot -->
      <template #item.actions="{ item }">
        <v-btn
          slot="activator"
          icon
          :to="{
            name: 'main-observation-observations-edit',
            params: { id: item.id },
          }"
        >
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
      </template>
    </v-data-table>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { readAdminObservations } from "@/store/admin/getters";
import { dispatchGetObservations } from "@/store/admin/actions";

@Component
export default class ManageObservation extends Vue {
  public headers = [
    {
      text: "Description",
      sortable: true,
      value: "description",
      align: "left",
    },
    {
      text: "Observation Type",
      sortable: true,
      value: "observation_type",
      align: "left",
    },
    {
      text: "Photo",
      sortable: true,
      value: "photo",
      align: "left",
    },
    {
      text: "Date",
      sortable: true,
      value: "date",
      align: "left",
    },
    {
      text: "Coordinates Lat",
      sortable: true,
      value: "coordinates.lat",
      align: "left",
    },
    {
      text: "Coordinates Lng",
      sortable: true,
      value: "coordinates.lng",
      align: "left",
    },
    {
      text: "Actions",
      value: "actions",
      sortable: false,
    },
  ];

  get observations() {
    return readAdminObservations(this.$store);
  }

  public async mounted() {
    await dispatchGetObservations(this.$store);
  }
}
</script>
