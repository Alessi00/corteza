<template>
  <div>
    <!-- Configure source module -->
    <div>
      <div>
        <b-form-group>
          <b-form-select
            v-model="moduleID"
            :options="modules"
            text-field="name"
            class="mt-1"
            value-field="moduleID"
          >
            <template slot="first">
              <option
                :value="null"
                disabled
              >
                {{ $t('chart.edit.modulePick') }}
              </option>
            </template>
          </b-form-select>
        </b-form-group>
      </div>
    </div>

    <!-- Configure report filters -->
    <div
      v-if="!!module"
      class="mt-1"
    >
      <div class="border p-2 mb-2">
        <h5 class="mb-3">
          {{ $t('chart.edit.filter.label') }}
        </h5>
        <b-form-group>
          <b-form-select
            v-model="report.filter"
            :disabled="customFilter"
            :options="predefinedFilters"
          >
            <template slot="first">
              <option :value="''">
                {{ $t('chart.edit.filter.noFilter') }}
              </option>
            </template>
          </b-form-select>

          <b-form-checkbox
            v-model="customFilter"
            class="mt-1"
          >
            {{ $t('chart.edit.filter.customize') }}
          </b-form-checkbox>

          <b-form-textarea
            v-if="customFilter"
            v-model="report.filter"
            placeholder="a = 1 AND b > 2"
          />
        </b-form-group>
      </div>
    </div>

    <slot
      name="y-axis"
      :report="editReport"
    />

    <!-- Configure report dimensions -->
    <div v-if="!!module">
      <div class="border p-2 mb-2">
        <fieldset
          v-for="(d, i) in dimensions"
          :key="i"
        >
          <h5 class="mb-3">
            {{ $t('chart.edit.dimension.label') }}
          </h5>

          <b-form-group
            horizontal
            :label-cols="2"
            breakpoint="md"
            :label="$t('chart.edit.dimension.fieldLabel')"
          >
            <b-form-select
              v-model="d.field"
              @change="onDimFieldChange($event, d)"
              :options="dimensionFields"
              text-field="name"
              value-field="name"
            >
              <template slot="first">
                <option
                  disabled
                  :value="undefined"
                >
                  {{ $t('chart.edit.dimension.fieldPlaceholder') }}
                </option>
              </template>
            </b-form-select>
          </b-form-group>

          <b-form-group
            horizontal
            :label-cols="2"
            breakpoint="md"
            :label="$t('chart.edit.dimension.function.label')"
          >
            <b-form-select
              v-model="d.modifier"
              :disabled="!d.field || !isTemporalField(d.field)"
              :options="dimensionModifiers"
            >
              <template slot="first">
                <option
                  disabled
                  :value="undefined"
                >
                  {{ $t('chart.edit.dimension.function.placeholder') }}
                </option>
              </template>
            </b-form-select>
          </b-form-group>

          <template v-if="!unSkippable">
            <b-form-group
              horizontal
              :label-cols="2"
              breakpoint="md"
            >
              <b-form-checkbox
                v-model="d.skipMissing"
              >
                {{ $t('chart.edit.dimension.skipMissingValues') }}
              </b-form-checkbox>
            </b-form-group>

            <b-form-group
              v-if="!d.skipMissing"
              horizontal
              :label-cols="2"
              breakpoint="md"
              :label="$t('chart.edit.dimension.defaultValueLabel')"
              :description="$t('chart.edit.dimension.defaultValueFootnote')"
            >
              <b-form-input
                v-model="d.default"
                :type="defaultValueInputType(d)"
              />
            </b-form-group>
          </template>

          <slot
            name="dimension-options"
            :dimension="d"
            :field="getField(d)"
          />
        </fieldset>
      </div>

      <!-- Configure report metrics -->
      <draggable
        class="metrics border px-3 py-2"
        :list.sync="metrics"
        :options="{ group: 'metrics_'+moduleID, sort: true }"
      >
        <fieldset
          v-for="(m,i) in metrics"
          :key="i"
          class="main-fieldset"
        >
          <font-awesome-icon
            class="align-baseline text-secondary mr-2"
            v-if="metrics.length>1"
            :icon="['fas', 'grip-vertical']"
          />
          <h5 class="mb-3 d-inline-block">
            {{ $t('chart.edit.metric.label') }}
          </h5>
          <b-button
            v-if="metrics.length > 1"
            @click.prevent="removeMetric(i)"
            variant="link"
            class="text-danger align-baseline"
          >
            <font-awesome-icon :icon="['far', 'trash-alt']" />
          </b-button>

          <b-form-group
            horizontal
            :label-cols="2"
            breakpoint="md"
            :label="$t('chart.edit.metric.fieldLabel')"
          >
            <b-form-select
              v-model="m.field"
              :options="metricFields"
              text-field="name"
              value-field="name"
            >
              <template slot="first">
                <option
                  disabled
                  :value="undefined"
                >
                  {{ $t('chart.edit.metric.fieldPlaceholder') }}
                </option>
              </template>
            </b-form-select>
          </b-form-group>

          <b-form-group
            horizontal
            :label-cols="2"
            breakpoint="md"
            :label="$t('chart.edit.metric.function.label')"
          >
            <b-form-select
              v-model="m.aggregate"
              :disabled="!m.field || m.field === 'count'"
              :options="metricAggregates"
            >
              <template slot="first">
                <option
                  disabled
                  :value="undefined"
                >
                  {{ $t('chart.edit.metric.function.placeholder') }}
                </option>
              </template>
            </b-form-select>
          </b-form-group>

          <slot
            name="metric-options"
            :metric="m"
          />

        </fieldset>
      </draggable>
    </div>

    <b-button
      v-if="canAddMetric"
      @click.prevent="addMetric"
      variant="link"
      class="float-right"
    >
      + {{ $t('chart.edit.metric.add') }}
    </b-button>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import { compose } from '@cortezaproject/corteza-js'
import base from './base'

const aggregateFunctions = [
  {
    value: 'COUNTD',
    text: 'countd',
  },
  {
    value: 'SUM',
    text: 'sum',
  },
  {
    value: 'MAX',
    text: 'max',
  },
  {
    value: 'MIN',
    text: 'min',
  },
  {
    value: 'AVG',
    text: 'avg',
  },
  {
    value: 'STD',
    text: 'std',
  },
]

export default {
  name: 'ReportEdit',

  components: {
    draggable,
  },

  extends: base,

  data () {
    return {
      customFilter: false,

      metricAggregates: aggregateFunctions.map(af => ({ ...af, text: this.$t(`chart.edit.metric.function.${af.text}`) })),
      dimensionModifiers: compose.chartUtil.dimensionFunctions.map(df => ({ ...df, text: this.$t(`chart.edit.dimension.function.${df.text}`) })),
      predefinedFilters: compose.chartUtil.predefinedFilters.map(pf => ({ ...pf, text: this.$t(`chart.edit.filter.${pf.text}`) })),
    }
  },

  computed: {
    defaultValueInputType () {
      return ({ field }) => field === 'created_at' || (this.module.fields.filter(f => f.name === field)[0] || {}).kind === 'DateTime' ? 'date' : 'text'
    },

    canAddMetric () {
      return this.supportedMetrics < 0 || this.metrics.length < this.supportedMetrics
    },

    module () {
      return this.modules.find(m => m.moduleID === this.moduleID)
    },

    metricFields () {
      return [{ name: 'count' }, ...this.module.fields.filter(f => f.kind === 'Number').sort((a, b) => a.name.localeCompare(b.name))]
    },

    dimensionFields () {
      return [{ name: 'created_at', label: 'Created At', kind: 'DateTime' }].concat(this.module.fields).map(f => {
        const { name, label, kind, options } = f
        let disabled = !this.dimensionFieldKind.includes(kind)
        if (kind === 'String') {
          // this removes the need to check if we allowed strings in the first place
          disabled = disabled || (options.useRichTextEditor || options.multiLine)
        }
        return { name, label, disabled }
      }).sort((a, b) => a.name.localeCompare(b.name))
    },

    moduleID: {
      get () {
        return this.report.moduleID
      },

      set (v) {
        this.report.moduleID = v
        this.$emit('update:report', { ...this.report, moduleID: v })
      },
    },

    colorScheme: {
      get () {
        return this.report.colorScheme
      },

      set (v) {
        this.report.colorScheme = v
        this.$emit('update:report', { ...this.report, colorScheme: v })
      },
    },

    metrics: {
      get () {
        return this.report.metrics
      },

      set (v) {
        this.report.metrics = v
        this.$emit('update:report', { ...this.report, metrics: v })
      },
    },

    dimensions: {
      get () {
        return this.report.dimensions
      },

      set (v) {
        this.$emit('update:report', { ...this.report, dimensions: v })
      },
    },
  },

  watch: {
    'report.filter': {
      handler: function (v) {
        // !! is required, since :disabled="..." marks the field as disabled if '' is provided
        console.log({
          v,
          xx: compose.chartUtil.predefinedFilters,
        })
        this.customFilter = (!!v && !!compose.chartUtil.predefinedFilters.find(({ value }) => value === v)) ||
          (!!v)
      },
      immediate: true,
    },
  },

  methods: {
    hasRelativeDisplay: compose.chartUtil.hasRelativeDisplay,

    getField ({ field }) {
      if (!field || !this.module) {
        return undefined
      }

      return this.module.fields.find(({ name }) => name === field)
    },

    addMetric () {
      this.metrics = this.metrics.concat([{}])
    },

    onDimFieldChange (f, d) {
      if (!this.isTemporalField(f)) {
        this.$set(d, 'modifier', this.dimensionModifiers[0].value)
      }
    },

    removeMetric (i) {
      this.metrics.splice(i, 1)
    },

    isTemporalField (name) {
      if (name === 'created_at') {
        return true
      }

      return !!this.module.fields.find(f => f.name === name && f.kind === 'DateTime')
    },
  },
}
</script>
