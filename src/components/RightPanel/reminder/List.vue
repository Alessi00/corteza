<template>
  <div>
    <h5 class="sticky-top bg-white p-1 mb-0">
      {{ $t('general.reminder.listLabel') }}
    </h5>

    <!-- Active reminders -->
    <b-form-group class="p-2 mb-0">
      <b-form v-for="r in remindersActive"
              :key="r.reminderID"
              class="checkbox d-flex align-items-center py-1 border-bottom text-justify">
        <font-awesome-icon :icon="['far', 'square']" class="text-secondary"></font-awesome-icon>
        <font-awesome-icon :icon="['fas', 'check']" class="text-primary"></font-awesome-icon>
        <div class="flex-grow-1 text-dark d-flex align-items-center reminder">
          <b-form-checkbox size="sm"
                           class="checkbox"
                           @change="dismiss($event, r)">
          {{ r.payload.title }}
          </b-form-checkbox>
        </div>
        <font-awesome-icon v-if="r.snoozeCount"
                           :icon="['far', 'clock']"
                           class="ml-2 small" />

        <font-awesome-icon v-else-if="r.remindAt"
                           :icon="['far', 'bell']"
                           class="ml-2 small"
                           v-b-tooltip.hover
                           :title="makeTooltip(r)" />
        <span v-else class="pr-3"> </span>

        <!-- @note currently, edit is disabled -->
        <!-- <b-button variant="link"
                  class="p-0"
                  @click="$emit('edit', r)">

          <font-awesome-icon :icon="['far', 'edit']" class="text-primary" />
        </b-button> -->
      </b-form>
    </b-form-group>

    <!-- Dismissed reminders -->
    <b-list-group class="p-2 pb-5">
      <div v-for="r in remindersDismissed"
              :key="r.reminderID"
              class="d-flex align-items-center">
        <font-awesome-icon :icon="['fas', 'check']" class="text-primary mr-1"></font-awesome-icon>
        <s class="text-secondary small flex-grow-1 py-1 pr-3 border-bottom text-justify">{{ r.payload.title }}</s>
      </div>
    </b-list-group>

    <div class="position-sticky text-center bg-white py-1 fixed-bottom">
      <b-button @click="$emit('edit')"
                size="sm">
        + {{ $t('general.reminder.add') }}
      </b-button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    reminders: {
      type: Array,
      required: true,
      default: () => [],
    },
  },

  computed: {
    remindersActive () {
      return this.reminders
        .filter(({ dismissedAt }) => !dismissedAt)
        .sort(this.stdSort)
    },
    remindersDismissed () {
      return this.reminders
        .filter(({ dismissedAt }) => dismissedAt)
        .sort(this.stdSort)
    },
  },

  methods: {
    dismiss (checked, r) {
      if (checked) {
        this.$emit('dismiss', r)
      }
    },

    stdSort (a, b) {
      if (!a.remindAtTime) {
        return -1
      }
      if (!b.remindAtTime) {
        return 0
      }

      return a.remindAtTime.diff(b.remindAtTime, 'ms')
    },

    makeTooltip ({ remindAtTime }) {
      return remindAtTime.format('DD. MM. YYYY hh:mm')
    },
  },
}
</script>

<style lang="scss" scoped>
.checkbox {
  .fa-check {
    opacity: 0;
    margin-right: -14px;
    margin-left: -14px;
  }

  &:hover {
    .fa-check {
      opacity: 1;
      transition: opacity 0.1s;
    }

    .fa-square {
      opacity: 0;
      transition: opacity 0.1s;
    }
  }

  /deep/ .custom-control-label {
    &::before,
    &::after {
      display: none;
      background: none;
    }
  }
}
</style>