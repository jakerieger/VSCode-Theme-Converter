<template>
  <div id="main">
    Select Theme:
    <select>
      <option v-for="theme in themes" v-bind:key="theme.id">{{theme.name}}</option>
    </select>
    Convert To:
    <select>
      <option v-for="editor in editors" v-bind:key="editor.id">{{editor.name}}</option>
    </select>

    <div class="theme-colors-preview">
      <div class="token-preview" :style="{color: token.color}" v-for="token in tokenColors" v-bind:key="token.id">
        {{token.scope}}
      </div>
    </div>
  </div>
</template>

<script>
//import fs from 'fs'
import os from 'os'
import path from 'path'
import fs, {
  readdirSync
} from 'fs'
//import dJSON from 'dirty-json'

/**
 * Sorts an array of strings by alphabetical order (A->Z)
 * @param {string} a - First string
 * @param {string} b - Second string
 * @returns {boolean} A or B higher
 */
function sortStrings(a, b) {
  a = a.toLowerCase()
  b = b.toLowerCase()

  return (a < b) ? -1 : (a > b) ? 1 : 0
}

export default {
  name: 'Main',
  data() {
    return {
      themes: [],
      editors: [{
          id: 0,
          name: 'Visual Studio'
        },
        {
          id: 1,
          name: 'JetBrains'
        },
        {
          id: 2,
          name: 'Atom'
        },
        {
          id: 3,
          name: 'QtCreator'
        }
      ],
      selectedTheme: 0,
      tokenColors: []
    }
  },
  mounted() {
    this.getThemesOnSystem()
    this.getThemeTokens(this.themes[7].variants[0].data)
  },

  methods: {
    /**
     * Apparently VSCode themes are riddled with invalid JSON syntax. This function attempts to clean those up and
     * tries to parse the 'corrected' JSON. Works like 95% of the time
     */
    FixInvalidJson(jsonText) {
      let fixedText

      // Remove comments
      const regex = /[\s]\/\/(.)*/g
      fixedText = jsonText.replace(regex, "")

      // Remove trailing commas
      let finalText
      function recursiveReplace(input) {
        const regex = /,[\r\n](\s)*[\]}]/gm
        const match = regex.exec(input)

        if (match != null) {
          let replacePos = match.index
          let newText = input.substr(0, replacePos) + input.substr(replacePos + 1)

          recursiveReplace(newText)
        } else {
          finalText = input
          return
        }
      }

      recursiveReplace(fixedText)

      return JSON.parse(finalText)
    },

    /**
     * Gets a list of all the currently installed VSCode themes on the computer
     */
    getThemesOnSystem() {
      const homeDir = os.homedir()
      const extensionsDir = path.join(homeDir, '/.vscode/extensions/')

      const extensionsFolders = fs.readdirSync(extensionsDir, {
          withFileTypes: true
        })
        .filter(dirent => dirent.isDirectory())
        .map(dirent => path.join(extensionsDir, dirent.name))


      for (let i = 0; i < extensionsFolders.length; i++) {
        const packageJsonData = JSON.parse(fs.readFileSync(extensionsFolders[i] + '\\package.json', {
          encoding: "utf8"
        }))
        if (packageJsonData.categories.includes("Themes") && !extensionsFolders[i].toLowerCase().includes("icon")) {

          let variants = []

          if (fs.existsSync(extensionsFolders[i] + "\\themes")) {
            let themeJsonFolder = path.join(extensionsFolders[i] + "\\themes")
            let themeJsonFiles = readdirSync(themeJsonFolder, {
                withFileTypes: true
              })
              .filter(dirent => path.extname(dirent.name).toLowerCase() === ".json")
              .map(dirent => dirent.name)

            if (themeJsonFiles.length > 0) {
              for (let k = 0; k < themeJsonFiles.length; k++) {
                let variantPath = path.join(extensionsFolders[i], "\\themes\\", themeJsonFiles[k])
                let variantJson

                try {
                  variantJson = JSON.parse(fs.readFileSync(variantPath, { encoding: "utf8" }))
                } catch (error) {
                  console.warn(`Error parsing ${themeJsonFiles[k]}`, error, "\n[!] Attempting to correct invalid JSON")
                  try {
                    variantJson = this.FixInvalidJson(fs.readFileSync(variantPath, { encoding: "utf8" }))
                  } catch (error) {
                    console.error("Dirty json parsing failed. Ignoring theme", error)
                    continue
                  }
                }


                variants.push({
                  id: k,
                  name: variantJson.name,
                  path: variantPath,
                  data: variantJson
                })
              }
            }

          } else if (fs.existsSync(extensionsFolders[i] + "\\build")) {
            let themeJsonFolder = path.join(extensionsFolders[i] + "\\build")
            let themeJsonFiles = readdirSync(themeJsonFolder, {
                withFileTypes: true
              })
              .filter(dirent => path.extname(dirent.name).toLowerCase() === ".json")
              .map(dirent => dirent.name)

            if (themeJsonFiles.length > 0) {
              for (let k = 0; k < themeJsonFiles.length; k++) {
                let variantPath = path.join(extensionsFolders[i], "\\build\\", themeJsonFiles[k])
                let variantJson

                try {
                  variantJson = JSON.parse(fs.readFileSync(variantPath, { encoding: "utf8" }))
                } catch (error) {
                  console.warn(`Error parsing ${themeJsonFiles[k]}`, error, "\n[!] Attempting to correct invalid JSON")
                  try {
                    variantJson = this.FixInvalidJson(fs.readFileSync(variantPath, { encoding: "utf8" }))
                  } catch (error) {
                    console.error("Dirty json parsing failed. Ignoring theme", error)
                    continue
                  }
                  
                }

                variants.push({
                  id: k,
                  name: variantJson.name,
                  path: variantPath,
                  data: variantJson
                })
              }
            }

          } else {
            continue
          }


          this.themes.push({
            name: packageJsonData.displayName,
            path: extensionsFolders[i],
            variants
          })
        }
      }

      this.themes = this.themes.sort((a, b) => {
        return sortStrings(a.name, b.name)
      })
    },

    getThemeTokens(theme) {
      this.tokenColors = []

      for (let i = 0; i < theme.tokenColors.length; i++) {
        this.tokenColors.push({
          id: i,
          scope: theme.tokenColors[i].scope.toString().includes(",") ? theme.tokenColors[i].scope.toString().split(",")[0] : theme.tokenColors[i].scope.toString(),
          color: theme.tokenColors[i].settings.foreground
        })
      }
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@300&display=swap');

li {
  font-family: 'Fira Code', monospace;
}

.theme-colors-preview {
  padding: 10px;
  border-radius: 6px;
  background: #08080a;
  width: 66vw;
}

.token-preview {

}
</style>