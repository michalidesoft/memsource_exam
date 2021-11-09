<template>
  <v-card>
    <v-data-table
      :headers="headers"
      :items="projects"
      :loading="isLoading"
      :sort-by="['id', 'name','status','dueDate']"
      class="elevation-1"
      height="700"
      hide-default-footer
      item-key="id"
      loading-text="Loading... Please wait"
      multi-sort
      no-data-text="No data"
    >
      <!--    <template v-slot:top>-->
      <!--      <v-text-field-->
      <!--        v-model="search"-->
      <!--        label="Search (UPPER CASE ONLY)"-->
      <!--        class="mx-4"-->
      <!--      ></v-text-field>-->
      <!--    </template>-->
      <template v-slot:item.actions="{ item }">
        <v-icon
          class="mr-2"
          small
          @click="editItem(item)"
        >
          {{ icons.mdiPencil }}
        </v-icon>
        <v-icon
          small
          @click="deleteItem(item)"
        >
          {{ icons.mdiDelete }}
        </v-icon>
      </template>
      <template v-slot:footer>
        <v-row></v-row>
        <v-row></v-row>
        <v-row></v-row>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import { useQuery } from 'vue-query'
import axios from 'axios'
import { mdiDelete, mdiPencil } from '@mdi/js'

const URL = 'http://localhost:8090/projects'

async function getProjects() {
  const response = await axios(URL, {
    method: 'GET',
    mode: 'no-cors',
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json',
    },
    withCredentials: false,
    credentials: 'same-origin',
  })
  // eslint-disable-next-line no-underscore-dangle
  console.log(response.data._embedded.projects)

  // eslint-disable-next-line no-underscore-dangle
  return response.data._embedded.projects
}

export default {
  setup() {
    // const fetcher = () => fetch(URL).then(res => res.json())

    const {
      isLoading,
      isError,
      data: projects,
      error,
    } = useQuery(
      'projects',
      getProjects,
    )
    console.log(projects)

    return {
      isLoading,
      isError,
      projects,
      error,
      icons: {
        mdiDelete,
        mdiPencil,
      },
    }
  },
  computed: {

    headers() {
      return [
        {
          text: 'ID',
          align: 'start',
          sortable: true,
          value: 'id',
        },
        {
          text: 'name',
          sortable: true,
          value: 'name',
        },
        {
          text: 'sourceLanguage',
          sortable: true,
          value: 'sourceLanguage',
        },
        {
          text: 'Status ',
          sortable: true,
          value: 'status',
        },
        {
          text: 'targetLanguages',
          value: 'targetLanguages',
        },
        {
          text: 'Due data ',
          sortable: true,
          value: 'dueDate',
        },
        {
          text: 'Actions',
          value: 'actions',
          sortable: false,
        },
      ]
    },
  },

  methods: {
    // eslint-disable-next-line no-unused-vars
    editItem(item) {
      // this.editedIndex = this.desserts.indexOf(item)
      // this.editedItem = Object.assign({}, item)
      // this.dialog = true
    },

    // eslint-disable-next-line no-unused-vars
    deleteItem(item) {
      // this.editedIndex = this.desserts.indexOf(item)
      // this.editedItem = Object.assign({}, item)
      // this.dialogDelete = true
    },

    // filterOnlyCapsText(value, search, item) {
    //   return value != null
    //       && search != null
    //       && typeof value === 'string'
    //       && value.toString().toLocaleUpperCase().indexOf(search) !== -1
    // },
  },

  // const desserts = [
  //   {
  //     dessert: 'Frozen Yogurt',
  //     calories: 159,
  //     fat: 6,
  //     carbs: 24,
  //     protein: 4,
  //   },
  //   {
  //     dessert: 'Ice cream sandwich',
  //     calories: 237,
  //     fat: 6,
  //     carbs: 24,
  //     protein: 4,
  //   },
  //   {
  //     dessert: 'Eclair',
  //     calories: 262,
  //     fat: 6,
  //     carbs: 24,
  //     protein: 4,
  //   },
  //   {
  //     dessert: 'Cupcake',
  //     calories: 305,
  //     fat: 6,
  //     carbs: 24,
  //     protein: 4,
  //   },
  //   {
  //     dessert: 'Gingerbread',
  //     calories: 356,
  //     fat: 6,
  //     carbs: 24,
  //     protein: 4,
  //   },
  // ]
  //
  // return {
  //   desserts,
  // }

}
</script>
