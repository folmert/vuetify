<template>
  <v-snackbar
    v-model="value"
    :color="snackbar.color.value"
    :timeout="-1"
    bottom
    right
  >
    <div class="d-flex">
      <span
        v-if="snackbar.emoji"
        class="mr-2"
        v-text="snackbar.emoji"
      />

      <app-md
        class="mb-n4"
        v-text="snackbar.text"
      />
    </div>

    <template #action="{ attrs }">
      <v-btn
        class="mr-2"
        text
        v-bind="{ ...bind, ...attrs }"
        @click="onClick"
      >
        {{ snackbar.action_text }}
      </v-btn>

      <v-btn
        color="white"
        icon
        @click="value = false"
      >
        <v-icon small>
          $close
        </v-icon>
      </v-btn>
    </template>
  </v-snackbar>
</template>

<script>
  // Utilities
  import { get, sync } from 'vuex-pathify'
  import { localeLookup } from '@/i18n/util'

  export default {
    name: 'DefaultSnackbar',

    computed: {
      ...sync('snackbar', [
        'snackbar',
        'value',
      ]),
      ...sync('user', [
        'notifications',
        'last@notification',
      ]),
      locale: get('route/params@locale'),
      bind () {
        const { action: href } = this.snackbar

        return href.startsWith('http')
          ? { href, target: '_blank', rel: 'noopener' }
          : { to: `/${localeLookup(this.locale)}${href}` }
      },
    },

    watch: {
      snackbar (val) {
        if (!val.slug) return

        this.value = true
      },
      value (val) {
        if (val) return

        this.notifications.push(this.snackbar.slug)
        this.notification = Date.now()
      },
    },

    methods: {
      onClick () {
        this.value = false

        this.$gtag.event('click', {
          event_category: 'notification',
          event_label: 'snackbar',
          value: this.snackbar.slug,
        })
      },
    },
  }
</script>

<style>
  .snack-markdown p {
    margin-bottom: 0 !important;
  }
</style>
