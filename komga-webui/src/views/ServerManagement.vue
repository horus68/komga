<template>
  <v-container fluid class="pa-6">
    <v-row>
      <v-col><span class="text-h5">{{ $t('server.server_management.section_title') }}</span></v-col>
    </v-row>
    <v-row>
      <v-col cols="auto">
        <v-btn @click="downloadLogFile"
        >{{ $t('server.server_management.download_log') }}
        </v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="auto">
        <v-btn @click="scanAllLibraries(false)">{{ $t('server.server_management.button_scan_libraries') }}</v-btn>
      </v-col>
      <v-col cols="auto">
        <v-btn @click="scanAllLibraries(true)"
               color="warning"
        >{{ $t('server.server_management.button_scan_libraries_deep') }}
        </v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="auto">
        <v-btn @click="confirmEmptyTrash = true">{{ $t('server.server_management.button_empty_trash') }}</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="auto">
        <v-btn @click="cancelAllTasks"
               color="warning"
        >{{ $t('server.server_management.button_cancel_all_tasks') }}
        </v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="auto">
        <v-btn @click="modalStopServer = true"
               color="error"
        >{{ $t('server.server_management.button_shutdown') }}
        </v-btn>
      </v-col>
    </v-row>

    <confirmation-dialog
      v-model="confirmEmptyTrash"
      :title="$t('dialog.empty_trash.title')"
      :body="$t('dialog.empty_trash.body')"
      :button-confirm="$t('dialog.empty_trash.button_confirm')"
      @confirm="emptyTrash"
    />

    <confirmation-dialog
      v-model="modalStopServer"
      :title="$t('dialog.server_stop.dialog_title')"
      :body="$t('dialog.server_stop.confirmation_message')"
      :button-confirm="$t('dialog.server_stop.button_confirm')"
      button-confirm-color="error"
      @confirm="stopServer"
    />

  </v-container>
</template>

<script lang="ts">
import Vue from 'vue'
import ConfirmationDialog from '@/components/dialogs/ConfirmationDialog.vue'
import {ERROR, ErrorEvent, NOTIFICATION, NotificationEvent} from '@/types/events'
import {LibraryDto} from '@/types/komga-libraries'
import jsFileDownloader from 'js-file-downloader'
import urls from '@/functions/urls'

export default Vue.extend({
  name: 'ServerManagement',
  components: {ConfirmationDialog},
  data: () => ({
    modalStopServer: false,
    confirmEmptyTrash: false,
  }),
  computed: {
    libraries(): LibraryDto[] {
      return this.$store.state.komgaLibraries.libraries
    },
  },
  methods: {
    emptyTrash() {
      this.libraries.forEach(library => {
        this.$komgaLibraries.emptyTrash(library)
      })
    },
    scanAllLibraries(scanDeep: boolean) {
      this.libraries.forEach(library => {
        this.$komgaLibraries.scanLibrary(library, scanDeep)
      })
    },
    async cancelAllTasks() {
      const count = await this.$komgaTasks.deleteAllTasks()
      this.$eventHub.$emit(NOTIFICATION, {
        message: this.$tc('server.server_management.notification_tasks_cancelled', count),
      } as NotificationEvent)
    },
    async stopServer() {
      try {
        await this.$actuator.shutdown()
      } catch (e) {
        this.$eventHub.$emit(ERROR, {message: e.message} as ErrorEvent)
      }
    },
    downloadLogFile() {
      new jsFileDownloader({
        url: `${urls.originNoSlash}${this.$actuator.logfileUrl()}`,
        filename: 'komga.log',
        withCredentials: true,
        forceDesktopMode: true,
      })
    },
  },
})
</script>

<style scoped>

</style>
