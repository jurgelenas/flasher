<template>
      <v-container>

        <v-sheet class="pt-1 pl-1 rounded">
          <p class="h6 font-weight-bold pt-1 pl-2">Flasher Settings</p>
          <v-sheet class="pl-5">
            <v-switch
              v-model="advancedFlash"
              inset
              label="Advanced Mode (Development & unstable versions available)"
              @change="updateRSProps()"
            ></v-switch>
          </v-sheet>
        </v-sheet>

        <br>

        <v-sheet class="pt-1 pl-1 rounded">
          <p class="h6 font-weight-bold pt-1 pl-2">Backup Settings</p>
          <v-sheet class="pl-5 pr-6 pb-6">
            <v-text-field
              v-model="defaultDir"
              label="Default Backup Directory..."
              hide-details="auto"
              @change="updateProps()"
            ></v-text-field>
          </v-sheet>
        </v-sheet>

        <br>

        <v-sheet class="pt-1 pl-1 rounded">
          <p class="h6 font-weight-bold pt-1 pl-2">App Options</p>
          <v-sheet class="pl-5">
            <v-switch
              v-model="themeSwitch"
              inset
              label="Dark Mode"
              @change="updateTheme()"
            ></v-switch>
          </v-sheet>
        </v-sheet>

        <br>

        <v-sheet class="pt-1 pl-1 rounded">
          <p class="h6 font-weight-bold pt-1 pl-2">Logging</p>
          <v-sheet class="pl-5 pr-5 pb-5">
            <v-textarea
              v-model="logText"
              readonly
              style="font-weight: bold;"
              no-resize
            ></v-textarea>

            <v-btn
              class="mt-4"
              color="primary"
              block
              elevation="2"
              large
              @click="clearLog"
            >Clear Log</v-btn>
            <v-btn
              class="mt-4"
              color="primary"
              block
              elevation="2"
              large
              @click="copyClipboard"
            >Copy to clipboard</v-btn>
          </v-sheet>
        </v-sheet>

        <v-dialog
          v-model="dialog"
          hide-overlay
          persistent
          width="142"
        >
          <v-card
            color="primary"
            dark
          >
            <v-card-text class="pt-2">
              Relaunching...
              <br>
              <br>
              <v-progress-circular
                :rotate="-90"
                :size="92"
                :width="10"
                :value="value"
                color="white"
              >
                {{ value / 20 }}
              </v-progress-circular>
            </v-card-text>
          </v-card>
        </v-dialog>

        <br>
      </v-container>
</template>

<script>
const tmplog = require("../support/tmplog.js");
const {clipboard} = require('electron');
const {app} = require('electron').remote;

export default {
  name: 'SettingsPage',

  data () {
      return {
        advancedFlash: this.$store.getters.getOptions.advancedFlash,
        themeSwitch: this.$store.getters.getOptions.themeSwitch,
        defaultDir: this.$store.getters.getOptions.defaultDir,
        logText: tmplog.getFormattedLog(),

        dialog: false,
        value: 100,
        interval: {}
      }
  },

  methods: {
    updateTheme() {
      this.$vuetify.theme.dark = this.themeSwitch;
      //this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
      this.updateProps()
    },

    copyClipboard() {
      clipboard.writeText(this.logText, 'selection')
    },

    updateRSProps() {
      this.updateProps();

      this.dialog = true;
      this.interval = setInterval(() => {
        if (this.value == 0) {
          app.relaunch();
          app.exit();
        }
        this.value -= 20
      }, 1000)
    },

    updateProps() {
      this.$store.commit('update', {
        advancedFlash: this.advancedFlash,
        themeSwitch: this.themeSwitch,
        defaultDir: this.defaultDir
      })
    },

    clearLog() {
      tmplog.setLog([]);
      this.logText = tmplog.getFormattedLog();
    }
  }
}

</script>