<template>
  <div id="app">
    <div class="appElement">
      <h2>Locale Editor</h2>
    </div>
    <div class="appElement">
      <button
        style="margin-right: 20px;"
        @click="reset">
        Reset
      </button>
      <button
        @click.prevent="download">
        Download
      </button>
      <br><br>
      Load from file:
      <input
        ref="fileUpload"
        type="file"
        @change="upload">
      <br>
      Load from URL:
      <input
        v-model="fileUrl"
        type="text"
        size="32">
      <button
        @click.prevent="loadFromUrl">
        Load
      </button>
    </div>
    <div class="appElement">
      <br>New language:
      <input
        ref="newLanguage"
        type="text"
        @keyup.enter="newLanguage">
      <button
        @click="newLanguage">
        Add
      </button>
      <br><br>
    </div>
    <div
      id="localeTable"
      ref="localeTable">
      <FlexibleTable
        :items="items"
        :fields="fields">
        <template
          v-for="language in getLanguages(locale)"
          :slot="language"
          slot-scope="{ value }">
          <input
            :key="language"
            type="text"
            style="width: 90%;"
            :value="value.value"
            @keyup="set(language, value.path, $event)">
        </template>
      </FlexibleTable>
    </div>
    <div class="appElement">
      <br>New path:
      <input
        ref="newPath"
        type="text"
        @keyup.enter="newPath">
      <select
        ref="newPathLanguage"
        @keyup.enter="newPath">
        <option
          v-for="language in getLanguages(locale)"
          :key="language"
          :value="language">
          {{ language }}
        </option>
      </select>
      <button
        @click="newPath">
        Add
      </button>
    </div>
    <div
      v-if="loading"
      id="loading">
      <div>Loading...</div>
    </div>
  </div>
</template>

<script>
import _ from "lodash"
import axios from "axios"
import FlexibleTable from "vue-flexible-table"
import FileSaver from "file-saver"

export default {
  name: "App",
  components: {
    FlexibleTable
  },
  data() {
    return {
      locale: {
        en: {
          hello: "Hello",
        },
        de: {
          hello: "Hallo",
        }
      },
      fileUrl: "",
      loading: false,
    }
  },
  computed: {
    localeTransformed() {
      let locale = {}
      _.forOwn(this.locale, (value, language) => {
        for (let path of this.getPaths(value)) {
          _.set(locale, `${path}.${language}`, _.get(value, path))
        }
      })
      return locale
    },
    items() {
      let items = []
      for (let path of this.getPaths(this.locale)) {
        let item = {
          path
        }
        for (let language of this.getLanguages(this.locale)) {
          item[language] = {
            value: _.get(this.locale, `${language}.${path}`, ""),
            path
          }
        }
        items.push(item)
      }
      return items
    },
    fields() {
      let fields = [
        {
          key: "path",
          width: "20%",
          minWidth: "200px",
          align: "left",
        }
      ]
      for (let language of this.getLanguages(this.locale)) {
        fields.push({
          key: language,
          label: language,
          width: "20%",
          minWidth: "150px",
          align: "center",
        })
      }
      return fields
    },
  },
  watch: {
    fileUrl() {
      let query = {}
      if (this.fileUrl) {
        query.fromUrl = this.fileUrl
      }
      this.$router.push({ query })
    },
  },
  mounted() {
    if (this.$route.query.fromUrl) {
      this.fileUrl = this.$route.query.fromUrl
      this.loadFromUrl()
    }
  },
  methods: {
    getPaths(locale) {
      let paths = []
      for (let object of Object.values(locale)) {
        paths = _.union(paths, this._getPaths(object))
      }
      return paths
    },
    _getPaths(object, path = "") {
      let paths = []
      _.forOwn(object, (value, key) => {
        let newPath = `${path}${key}`
        if (_.isString(value)) {
          if (!paths.includes(newPath)) {
            paths.push(newPath)
          }
        } else {
          paths = paths.concat(this._getPaths(value, newPath + "."))
        }
      })
      return paths
    },
    getLanguages(locale) {
      return Object.keys(locale)
    },
    upload(event) {
      let reader = new FileReader()
      reader.onload = event => {
        this.locale = JSON.parse(event.target.result) || {}
        this.$refs.fileUpload.value = ""
      }
      reader.readAsText(event.target.files[0])
    },
    set(language, path, event) {
      _.set(this.locale, `${language}.${path}`, event.target.value)
    },
    download() {
      var blob = new Blob([JSON.stringify(this.locale, null, 2)], {type: "text/plain;charset=utf-8"})
      FileSaver.saveAs(blob, "locale.json")
    },
    newLanguage() {
      let language = this.$refs.newLanguage.value
      if (language && !this.locale[language]) {
        this.$set(this.locale, language, {})
      }
      this.$refs.newLanguage.value = ""
    },
    newPath() {
      let language = this.$refs.newPathLanguage.value
      let path = this.$refs.newPath.value
      let paths = this.getPaths(this.locale).map(value => `${language}.${value}`)
      if (path && language && this.getLanguages(this.locale).includes(language) && !this.getPaths(this.locale).includes(path)) {
        let pathList = [language].concat(path.split("."))
        let currentPath = null, i = 0
        while(i < pathList.length) {
          let previousPath = currentPath
          let currentKey = pathList[i]
          currentPath = (currentPath ? currentPath + "." : "") + pathList[i]
          if (paths.includes(currentPath)) {
            console.warn("Part of paths already exists!", currentPath)
            currentPath = null
            break
          } else if (!_.get(this.locale, currentPath)) {
            this.$set(_.get(this.locale, previousPath), currentKey, {})
          }
          i += 1
        }
        if (currentPath) {
          _.set(this.locale, currentPath, "")
          this.$refs.newPath.value = ""
          this.$refs.localeTable.scrollTop = 100000
        }
      } else {
        console.warn("Invalid input.")
      }
    },
    reset() {
      this.locale = {}
    },
    loadFromUrl() {
      if (this.fileUrl == "" || !this.fileUrl.startsWith("http")) {
        return
      }
      this.loading = true
      axios.get(this.fileUrl).then(response => response.data).then(data => {
        this.locale = data
      }).catch(error => {
        alert("Error when loading from URL:", error)
      }).finally(() => {
        this.loading = false
      })
    },
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  flex-direction: column;
  height: 95vh;
}
.appElement {
  flex: none;
}
#localeTable {
  flex: 1;
  height: 0;
  min-height: 200px;
  overflow: scroll;
}

#loading {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(255, 255, 255, 0.5);
  display:flex;
  justify-content:center;
  align-items:center;
}
#loading > div {
  font-weight: 700;
  font-size: 2rem;
}

/* Overwrite the default to keep the scrollbar always visible */
::-webkit-scrollbar {
  -webkit-appearance: none;
  width: 9px;
  height: 9px;
}
::-webkit-scrollbar-thumb {
  border-radius: 6px;
  box-shadow: inset 0 0 5px 5px #AEAEAE;
  border: solid 1px transparent;
}
</style>
