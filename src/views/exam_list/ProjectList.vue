<template>
  <v-card>
    <v-data-table
      :custom-sort="customSort"
      :headers="headers"
      :items="projects"
      :loading="isLoading"
      class="elevation-1"
      dense
      height="400"
      hide-default-footer
      item-key="id"
      loading-text="Loading... Please wait"
      multi-sort
      no-data-text="No data"
    >
      <template v-slot:item.dateDue="{ item }">
        <span>{{ new Date(item.dateDue).toLocaleString() }}</span>
      </template>
      <template v-slot:item.dateCreated="{ item }">
        <span>{{ new Date(item.dateCreated).toLocaleString() }}</span>
      </template>
      <template v-slot:top>
        <v-toolbar
          flat
        >
          <v-toolbar-title>Projects list</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog
            v-model="dialog"
            max-width="500px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                class="mb-2"
                color="primary"
                dark
                v-bind="attrs"
                v-on="on"
              >
                New project
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                      cols="12"
                      md="12"
                      sm="6"
                    >
                      <v-text-field
                        v-model="editedItem.name"
                        label="Project name"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col
                      cols="12"
                      md="12"
                      sm="6"
                    >
                      <v-select
                        v-model="editedItem.source"
                        :items="languages"
                        label="Source language"
                      ></v-select>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col
                      cols="12"
                      md="12"
                      sm="6"
                    >
                      <v-select
                        v-model="editedItem.target"
                        :items="languages"
                        chips
                        filled
                        label="Target language"
                        multiple
                      ></v-select>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="close"
                >
                  Cancel
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="save"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog
            v-model="dialogDelete"
            max-width="500px"
          >
            <v-card>
              <v-card-title class="text-h5">
                Are you sure you want to delete this item?
              </v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="closeDelete"
                >
                  Cancel
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="deleteItemConfirm"
                >
                  OK
                </v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
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
        <div
          class="d-flex flex-column d-inline-flex"
          style="width: 100%"
        >
          <v-row v-if="projects !== undefined">
            <v-col
              cols="12"
              md="4"
              sm="6"
            >
              <v-list
                dense
              >
                <div class="ml-5">
                  Status
                </div>
                <v-list-item
                  v-for="(item, i) in Object.entries(countStatus())"
                  :key="i"
                >
                  <v-list-item-content> {{ item[0] }}: {{ item[1] }}</v-list-item-content>
                </v-list-item>
              </v-list>
            </v-col>
            <v-col
              cols="12"
              md="4"
              sm="6"
            >
              Most prominent language
              <br />
              {{ countSourceLanguage() }}
            </v-col>
            <v-col
              cols="12"
              md="4"
              sm="6"
            >
              Date due
              <br />
              {{ countDue() }}
            </v-col>
          </v-row>
        </div>
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
  return response.data._embedded.projects
}

export default {
  data: () => ({
    isEdit: false,
    dialog: false,
    dialogDelete: false,
    languages: ['cs', 'en', 'de', 'fi', 'zh', 'ru', 'hu', 'ja', 'ko', 'la'],
    editedItem: {
      name: '',
      sourceLanguage: [],
      targetLanguage: [],
    },
    defaultItem: {
      name: '',
      sourceLanguage: [],
      targetLanguage: [],
    },
  }),
  setup() {
    const {
      isLoading,
      isError,
      data: projects,
      error,
    } = useQuery(
      'projects',
      getProjects,
    )

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

    formTitle() {
      return this.isEdit ? 'Edit Item' : 'New Item'
    },
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
          text: 'Date due ',
          sortable: true,
          value: 'dateDue',
        },
        {
          text: 'Created ',
          value: 'dateCreated',
        },
        {
          text: 'Actions',
          value: 'actions',
          sortable: false,
        },
      ]
    }
    ,
  },

  methods: {
    countStatus() {
      // eslint-disable-next-line guard-for-in,no-restricted-syntax
      const result = {}

      // eslint-disable-next-line no-restricted-syntax
      for (const projectsKey of this.projects) {
        // eslint-disable-next-line no-prototype-builtins
        if (result.hasOwnProperty(projectsKey.status)) {
          // eslint-disable-next-line no-plusplus
          result[projectsKey.status]++
        } else {
          result[projectsKey.status] = 1
        }
      }

      return result
    },
    countSourceLanguage() {
      // eslint-disable-next-line guard-for-in,no-restricted-syntax
      const result = {}

      // eslint-disable-next-line no-restricted-syntax
      for (const projectsKey of this.projects) {
        // eslint-disable-next-line no-prototype-builtins
        if (result.hasOwnProperty(projectsKey.sourceLanguage)) {
          // eslint-disable-next-line no-plusplus
          result[projectsKey.sourceLanguage]++
        } else {
          result[projectsKey.sourceLanguage] = 1
        }
      }
      const sortedKey = Object.keys(result)
        .sort((a, b) => result[b] - result[a])

      return `${sortedKey[0]}: ${result[sortedKey[0]]}`
    },
    countDue() {
      return this.projects.filter(project => new Date(project.dateDue).getTime() <= Date.now()).length
    },

    customSort(items, index, isDesc) {
      items.sort((a, b) => {
        if (index === 'date') {
          if (!isDesc) {
            return new Date(a.date) - new Date(b.date)
          }

          return new Date(b.date) - new Date(a.date)
        }
        if (!isDesc) {
          return a[index] < b[index] ? -1 : 1
        }

        return b[index] < a[index] ? -1 : 1
      })

      return items
    },

    // eslint-disable-next-line no-unused-vars
    editItem(item) {
      this.editedItem.name = item.name
      this.editedItem.sourceLanguage = item.sourceLanguage
      this.editedItem.targetLanguage = item.targetLanguage

      // this.editedIndex = this.desserts.indexOf(item)
      // this.editedItem = Object.assign({}, item)
      this.isEdit = true
      this.dialog = true
    },

    // eslint-disable-next-line no-unused-vars
    deleteItem(item) {
      // this.editedIndex = this.desserts.indexOf(item)
      // this.editedItem = Object.assign({}, item)
      // this.dialogDelete = true
    },

    deleteItemConfirm() {
      // this.desserts.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close() {
      this.dialog = false

      // this.$nextTick(() => {
      //   this.editedItem = { ...this.defaultItem }
      //   this.editedIndex = -1
      // })
    },

    closeDelete() {
      this.dialogDelete = false

      // this.$nextTick(() => {
      //   this.editedItem = { ...this.defaultItem }
      //   this.editedIndex = -1
      // })
    },

    save() {
      if (this.isEdit) {
        // todo save with mutate and refresh list
      } else {
        // todo save with mutate and refresh
      }
      this.close()
    }
    ,

    // filterOnlyCapsText(value, search, item) {
    //   return value != null
    //       && search != null
    //       && typeof value === 'string'
    //       && value.toString().toLocaleUpperCase().indexOf(search) !== -1
    // },
  }
  ,
}
</script>
