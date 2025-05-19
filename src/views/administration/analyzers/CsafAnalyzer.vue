<template>
  <b-card no-body :header="header">
    <b-card-body>
      <h1>CSAF Analyzer</h1>
      <c-switch
        id="scannerEnabled"
        color="primary"
        v-model="scannerEnabled"
        label
        v-bind="labelIcon"
      />{{ $t('admin.analyzer_csaf_enable') }}
      <br />
      <!--<c-switch
        id="scannerCpeFuzzyEnableInput"
        color="primary"
        v-model="scannerCpeFuzzyEnableInput"
        label
        v-bind="labelIcon"
      />{{ $t('admin.analyzer_internal_fuzzy_enable') }}
      <br />
      <c-switch
        id="scannerCpeFuzzyExcludePurlInput"
        color="primary"
        v-model="scannerCpeFuzzyExcludePurlInput"
        label
        v-bind="labelIcon"
      />{{ $t('admin.analyzer_internal_fuzzy_exclude_purl') }}
      <br />
      <c-switch
        id="scannerCpeFuzzyExcludeInternalInput"
        color="primary"
        v-model="scannerCpeFuzzyExcludeInternalInput"
        label
        v-bind="labelIcon"
      />{{ $t('admin.analyzer_internal_fuzzy_exclude_internal') }}
    -->
      <br />
      <b-row>
        <b-col sm="2">
          <label for="threshold-input-range">Matching threshold:</label>
        </b-col>
        <b-col sm="3">
          <b-form-input
            id="threshold-input-range"
            type="range"
            v-model="thresholdValue"
          />
        </b-col>
        <b-col sm="2">
          <b-form-input
            id="threshold-input-number"
            type="number"
            v-model="thresholdValue"
          />
        </b-col>
      </b-row>
      
      <hr />
      {{ $t('admin.analyzer_csaf_desc') }}
    </b-card-body>
    <b-card-footer>
      <b-button variant="outline-primary" class="px-4" @click="saveChanges">{{
        $t('message.update')
      }}</b-button>
    </b-card-footer>
  </b-card>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import common from '../../../shared/common';
import configPropertyMixin from '../mixins/configPropertyMixin';

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String,
  },
  components: {
    cSwitch,
  },
  data() {
    return {
      scannerEnabled: false,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      thresholdValue: 80,
    };
  },
  methods: {
    saveChanges: function () {
      this.updateConfigProperties([
        {
          groupName: 'scanner',
          propertyName: 'csaf.enabled',
          propertyValue: this.scannerEnabled,
        },
        {
          groupName: 'scanner',
          propertyName: 'csaf.threshold',
          propertyValue: this.thresholdValue,
        },
      ]);
    },
  },
  created() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'scanner';
      });
      for (let i = 0; i < configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case 'csaf.enabled':
            this.scannerEnabled = common.toBoolean(item.propertyValue);
            break;
          case 'csaf.threshold':
            this.thresholdValue = item.propertyValue;
            break;
        }
      }
    });
  },
};
</script>
