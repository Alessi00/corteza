<template>
  <div>
    <b-form-group>
      <label
        class="d-block"
      >
        {{ $t('field.kind.record.moduleLabel') }}
      </label>
      <b-form-select
        v-model="f.options.moduleID"
        :options="modules"
        text-field="name"
        value-field="moduleID"
        class="form-control"
      >
        <template
          slot="first">
          <option
            disabled
            :value="undefined"
          >
            {{ $t('field.kind.record.modulePlaceholder') }}
          </option>
        </template>
      </b-form-select>
    </b-form-group>

    <b-form-group>
      <label
        class="d-block"
      >
        {{ $t('field.kind.record.recordFieldLabel') }}
      </label>
      <b-form-select
        v-model="f.options.labelField"
        class="form-control"
        :options="fields"
        :disabled="!module"
      >
        <template
          slot="first"
        >
          <option
            disabled
            :value="undefined"
          >
            {{ $t('field.kind.record.recordFieldPlaceholder') }}
          </option>
        </template>
      </b-form-select>
    </b-form-group>
    <b-form-group>
      <label
        class="d-block"
      >
        {{ $t('field.kind.record.queryFieldsLabel') }}
      </label>
      <b-form-select
        v-model="f.options.queryFields"
        class="form-control"
        :options="fields"
        multiple
        :disabled="!module"
      />
    </b-form-group>
    <b-form-group>
      <label
        class="d-block"
      >
        {{ $t('field.kind.record.prefilterLabel') }}
      </label>
      <b-form-textarea
        v-model="f.options.prefilter"
        class="form-control"
        :placeholder="$t('field.kind.record.prefilterPlaceholder')"
      />
      <b-form-text>
        <i18next path="field.kind.record.prefilterFootnote" tag="label">
          <code>${recordID}</code>
          <code>${ownerID}</code>
          <code>${userID}</code>
        </i18next>
      </b-form-text>
    </b-form-group>
    <b-form-group v-if="field.isMulti">
      <label
        class="d-block"
      >
        {{ $t('field.kind.select.optionType.label') }}
      </label>
      <b-form-radio-group
        v-model="f.options.selectType"
        :options="selectOptions"
        stacked
      />
    </b-form-group>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import base from './base'

export default {
  extends: base,

  data () {
    return {
      selected: null,
      selectOptions: [
        { text: this.$t('field.kind.select.optionType.default'), value: 'default' },
        { text: this.$t('field.kind.select.optionType.multiple'), value: 'multiple' },
        { text: this.$t('field.kind.select.optionType.each'), value: 'each' },
      ],
    }
  },

  computed: {
    ...mapGetters({
      modules: 'module/set',
    }),

    module () {
      if (this.field.options.moduleID !== '0') {
        return this.$store.getters['module/getByID'](this.field.options.moduleID)
      } else {
        return undefined
      }
    },

    fields () {
      const fields = this.module ? this.module.fields.map(f => { return { value: f.name, text: f.label || f.name } }) : []
      return fields.sort((a, b) => a.text.localeCompare(b.text))
    },
  },

  watch: {
    'field.options.moduleID' () {
      this.f.options.labelField = undefined
      this.f.options.queryFields = []
      this.f.options.selectType = 'default'
    },
  },
}
</script>
