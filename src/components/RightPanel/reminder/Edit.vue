<template>
  <div>
    <b-list-group-item class="flex-column align-items-start px-2 py-2 border-0">

      <b-button variant="link"
                size="sm"
                class="px-2"
                @click="$emit('cancel')">
        <font-awesome-icon :icon="['fas', 'chevron-left']"></font-awesome-icon>
        {{ $t('general.label.backWithoutSave') }}
      </b-button>
    </b-list-group-item>

    <b-list-group-item class="flex-column align-items-start px-2 py-2 border-0">
      <b-form @submit.prevent
              class="import-form">

        <b-form-group :label="$t('general.reminder.edit.titleLabel')">
          <b-form-input v-model="title"
                        required
                        type="text"
                        :placeholder="$t('general.reminder.edit.titlePlaceholder')" />

        </b-form-group>

        <b-form-group :label="$t('general.reminder.edit.notesLabel')">
          <b-form-textarea v-model="notes"
                           :placeholder="$t('general.reminder.edit.notesPlaceholder')"
                           rows="6"
                           max-rows="10" />

        </b-form-group>

        <b-form-group :label="$t('general.reminder.edit.remindAtLabel')">
          <b-form-select v-model="remindAt"
                         :options="remindAtPresets" />

        </b-form-group>

        <b-form-group :label="$t('general.reminder.edit.assigneeLabel')">
          <vue-select :options="assignees"
                      option-value="userID"
                      option-text="label"
                      v-model="assignedTo" />

        </b-form-group>

        <b-form-group v-if="reminder.payload.link"
                      :label="$t('general.reminder.routesTo')">

          <t-link :link="reminder.payload.link"
                  :resource="reminder.resource" />
        </b-form-group>
      </b-form>
    </b-list-group-item>
    <div class="position-sticky text-center bg-white py-1 fixed-bottom">
      <b-button variant="outline-primary"
                size="sm"
                class="px-2"
                @click="saveAndClose">
        {{ $t('general.label.saveAndClose') }}
      </b-button>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { VueSelect } from 'vue-select'
import { TLink } from 'corteza-webapp-common/src/components/Toaster/display'

export default {

  components: {
    VueSelect,
    TLink,
  },
  props: {
    edit: {
      type: Object,
      required: false,
      default: () => ({}),
    },

    myID: {
      type: String,
      required: true,
    },

    users: {
      type: Array,
      required: true,
      default: () => [],
    },
  },

  data () {
    return {
      // Do this, so we don't edit the original object
      reminder: {},
    }
  },

  computed: {
    title: {
      get: function () {
        return (this.reminder.payload || {}).title
      },
      set: function (v) {
        this.updPayload('title', v)
      },
    },

    notes: {
      get: function () {
        return (this.reminder.payload || {}).notes
      },
      set: function (v) {
        this.updPayload('notes', v)
      },
    },

    remindAt: {
      get: function () {
        return (this.reminder.payload || {}).remindAt || null
      },
      set: function (v) {
        this.updPayload('remindAt', v)
      },
    },

    assignedTo: {
      get: function () {
        return this.assignees.find(({ userID }) => userID === this.reminder.assignedTo)
      },
      set: function ({ userID }) {
        this.$set(this.reminder, 'assignedTo', userID)
      },
    },

    remindAtPresets () {
      return [
        { value: null, text: this.$t('general.reminder.edit.remindAtNone') },
        { value: 1000 * 60 * 15, text: this.$t('general.label.timeMinute', { t: 15 }) },
        { value: 1000 * 60 * 30, text: this.$t('general.label.timeMinute', { t: 30 }) },
        { value: 1000 * 60 * 60 * 1, text: this.$t('general.label.timeHour', { t: 1 }) },
        { value: 1000 * 60 * 60 * 2, text: this.$t('general.label.timeHour', { t: 2 }) },
        { value: 1000 * 60 * 60 * 24, text: this.$t('general.label.timeHour', { t: 24 }) },
      ]
    },

    assignees () {
      return this.users.map(({ userID, name: label }) => {
        if (userID === this.myID) {
          return { userID, label: this.$t('general.reminder.edit.assigneePlaceholder') }
        }
        return { userID, label }
      })
    },
  },

  watch: {
    edit: {
      handler: function () {
        this.reminder = JSON.parse(JSON.stringify(this.edit))
      },
      deep: true,
      immediate: true,
    },
  },

  methods: {
    saveAndClose () {
      // @todo support for updating times
      let r = {}
      if (this.remindAt) {
        r.remindAt = moment().add(this.remindAt, 'ms').toISOString()
      }

      r = {
        ...this.reminder,
        ...r,
      }

      this.$emit('save', r)
    },

    // Helper to handle undefined fields
    updPayload (f, v) {
      if (!this.reminder.payload) {
        this.$set(this.reminder, 'payload', {})
      }
      this.$set(this.reminder.payload, f, v)
    },
  },
}
</script>