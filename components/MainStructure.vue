<template>
  <div class="py-6">
    <div class="container is-fluid">
      <div class="columns">
        <div class="column is-2">
          <ArgentinaMap />
        </div>
        <div class="column is-10">
          <div class="">
            <div v-if="$fetchState.pending" class="column has-text-centered">
              <i class="fas fa-spin fa-5x fa-sync" />
            </div>
            <div v-else-if="$fetchState.error" class="column has-text-centered">
              <i class="fas fa-exclamation-triangle fa-5x fa-sync" /> Error!
            </div>
            <div v-else-if="chapter === null" class="has-text-centered">
              <!-- <h1 class="title is-4 has-text-primary has-text-centered is-700">
                Seleccione un Tema
              </h1> -->
              <div class="buttons is-centered mt-3">
                <button v-for="(chapterx,index) in chapters" :key="`button-chapter-${index}`" class="button is-outlined is-medium is-primary" :class="chapter == '' ? 'is-hidden' : null" @click="setChapter(chapterx)">
                  {{ chapterx }}
                </button>
              </div>
            </div>
            <div v-else>
              <h1 class="title is-1 has-text-primary has-text-centered is-700">
                {{ chapter }}
              </h1>
              <hr>
              <div class="columns is-multiline">
                <!-- <div v-for="graph in graphs" :key="`graph-${graph.id}`" class="column" :class="getColumns(graph)">
                  <StatsContainer :graph="graph" />
                </div> -->
                <StatsContainer v-for="graph in graphs" :key="`graph-${graph.id}`" class="column" :graph="graph" :class="getColumns(graph)" />
              </div>
            </div>
          </div>
          <hr class="my-6">
          <!-- <img src="@/assets/logo.png" class="image mx-auto my-6" width="400" alt=""> -->
          <img src="@/assets/logos.png" class="image mx-auto my-6" width="300" alt="">
          <!-- <img src="@/assets/flyer.png" class="image" alt=""> -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ArgentinaMap from './ArgentinaMap.vue'

export default {
  name: 'MainStructure',
  components: {
    ArgentinaMap
  },
  fetchOnServer: false,
  async fetch () {
    try {
      // console.log(app)
      // fetch with axios
      const response = await this.$axios.get(`https://sheets.googleapis.com/v4/spreadsheets/${this.googleSheetId}/values/index?key=${this.googleApiKey}`)
      const chapters = []
      const keys = response.data.values[0]
      const values = response.data.values.slice(1)
      const output = []
      values.forEach((entry) => {
        // eslint-disable-next-line prefer-const
        const graph = {}
        keys.forEach((k, i) => {
          if (k === 'cast_float') {
            graph[k] = entry[i].split(',')
          } else if (k === 'cast_int') {
            graph[k] = entry[i].split(',')
          } else if (k === 'include_nacional') {
            // console.log(k)
            // console.log(entry[i])
            // if entry[i] is "TRUE" save as boolean
            graph[k] = entry[i] === 'TRUE'
          } else if (k === 'habilitado') {
            // console.log(k)
            // console.log(entry[i])
            // if entry[i] is "TRUE" save as boolean
            graph[k] = entry[i] === 'TRUE'
          } else if (k === 'force_full_width') {
            // console.log(k)
            // console.log(entry[i])
            // if entry[i] is "TRUE" save as boolean
            graph[k] = entry[i] === 'TRUE'
          } else if (k === 'capitulo') {
            // chapters
            // if entry[i] is already in the array chapters, dont add it. If it is not, add it.
            if (!chapters.includes(entry[i])) {
              chapters.push(entry[i])
            }
            graph[k] = entry[i]
          } else {
            graph[k] = entry[i] !== '' ? entry[i] : null
          }
        })
        // create fake id with random number
        graph.id = `graph-${Math.floor(Math.random() * 1000000)}`
        graph.fetched = true
        output.push(graph)
      })
      this.$store.commit('data/setIndex', output)
      this.$store.commit('data/setChapters', chapters)
      this.$store.commit('data/setIsLoading', false)
    } catch (err) {
      this.$buefy.dialog.alert(`Error al obtener los datos: ${err.message}`)
      return null
    }
  },
  computed: {
    chapters () {
      return this.$store.state.data.chapters
    },
    chapter () {
      return this.$store.state.data.chapter
    },
    googleSheetId () {
      return process.env.googleSheetId
    },
    googleApiKey () {
      return process.env.googleApiKey
    },
    graphs () {
      // return this.$store.state.data.index
      return this.$store.getters['data/graphsChapter']
    }
  },
  methods: {
    getColumns (graph) {
      // if (graph.habilitado) {
      //   if (graph.force_full_width) {
      //     return 'is-12'
      //   } else {
      //     return `is-${this.$store.state.system.columns}`
      //   }
      // } else {
      //   return 'is-hidden'
      // }
      if (graph.force_full_width) {
        return 'is-12'
      } else {
        return `is-${this.$store.state.system.columns}`
      }
    },
    setChapter (chapter) {
      this.$gtag.event('change_chapter', {
        event_category: 'button',
        event_label: chapter
      })
      this.$store.commit('data/setChapter', chapter)
    }
  }
}
</script>

<style lang="scss" scoped>
.chart {
  min-height: 600px;
}
</style>
