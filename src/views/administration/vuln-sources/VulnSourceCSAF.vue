<template>
  <b-card no-body :header="header">
    <b-card-body>


      <!--TODO: Include logo file-->
      <img alt="CSAF logo" src="https://oasis-open.github.io/csaf-documentation/assets/images/screen-shot-2022-11-28-at-10.16.00-pm-279x97.png" width="65" />
      
      <hr />
      <c-switch
        color="primary"
        id="vulnsourceEnabled"
        label
        v-bind="labelIcon"
        v-model="vulnsourceEnabled"
        :disabled="enabledEcosystems.length === 0"
      />
      {{ $t('admin.vulnsource_csaf_advisories_enable') }}
      <hr />      
      <div>
        <b-card no-body>
          <b-tabs pills card>
            <b-tab title="CSAF Sources" active>
              <!--<b-card no-body :header="header">-->
              <b-card-body>
                <div id="repositoryToolbar" class="bs-table-custom-toolbar">
                  <b-button size="md" variant="outline-primary" v-b-modal.repositoryCreateRepositoryModal>
                    <span class="fa fa-plus"></span> {{ $t('admin.create_repository') }}
                  </b-button>
                </div>
                <bootstrap-table
                  ref="table"
                  :columns="columns"
                  :data="data"
                  :options="options"
                >
                </bootstrap-table>
              </b-card-body>
                <!--<repository-create-repository-modal
                  :type="type"
                  v-on:refreshTable="refreshTable"
                />-->
              <!--</b-card>-->
            </b-tab>
            <b-tab title="CSAF Documents"><b-card-text>List of retrieved csaf documents</b-card-text></b-tab>
          </b-tabs>
        </b-card>
      </div>
      <b-validated-input-group-form-input
        id="csaf-feeds-url"
        :label="$t('admin.vulnsource_csaf_base_url')"
        input-group-size="mb-3"
        rules="required"
        type="text"
        v-model="csafBaseUrl"
        lazy="true"
      />
      
      <hr />
      <b-form-group label="Ecosystems">
        <div class="list-group" style="width: 40%">
          <span v-for="ecosystem in enabledEcosystems" :key="ecosystem">
            <actionable-list-group-item
              :value="ecosystem"
              :delete-icon="true"
              @actionClicked="removeEcosystem(ecosystem)"
            />
          </span>
          <actionable-list-group-item
            :add-icon="true"
            @actionClicked="$root.$emit('bv::show::modal', 'ecosystemModal')"
          />
        </div>
      </b-form-group>
    </b-card-body>
    <b-card-footer>
      <b-button
        :disabled="this.vulnsourceEnabled && !this.csafBaseUrl"
        variant="outline-primary"
        class="px-4"
        @click="saveUrl"
      >
        {{ $t('message.update') }}
      </b-button>
    </b-card-footer>
    <ecosystem-modal v-on:selection="updateEcosystem" />
  </b-card>
</template>
<script>
import { Switch as cSwitch } from '@coreui/vue';
import common from '../../../shared/common';
import configPropertyMixin from '../mixins/configPropertyMixin';
import EcosystemModal from './EcosystemModal';
import ActionableListGroupItem from '../../components/ActionableListGroupItem.vue';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String,
  },
  components: {
    cSwitch,
    EcosystemModal,
    ActionableListGroupItem,
    BValidatedInputGroupFormInput,
  },
  data() {
    return {
      vulnsourceEnabled: false,
      aliasSyncEnabled: false,
      aliasGitHubIssueUrl: 'https://github.com/google/osv.dev/issues/888',
      csafBaseUrl: 'https://wid.cert-bund.de/.well-known/csaf-aggregator/aggregator.json', //TODO provide through system settings
      ecosystemConfig: null,
      enabledEcosystems: [],
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
    };
  },
  watch: {
    vulnsourceEnabled(toggleChange) {
      if (toggleChange === false) {
        this.enabledEcosystems = [];
        this.updateConfigProperties([
          {
            groupName: 'vuln-source',
            propertyName: 'google.osv.enabled',
            propertyValue: null,
          },
        ]);
      }
    },
  },
  methods: {
    removeEcosystem: function (ecosystem) {
      this.enabledEcosystems = this.enabledEcosystems.filter(
        (e) => e !== ecosystem,
      );
      this.vulnsourceEnabled = this.enabledEcosystems.length !== 0;
    },
    updateEcosystem: function (ecosystems) {
      this.$root.$emit('bv::hide::modal', 'ecosystemModal');
      for (let i = 0; i < ecosystems.length; i++) {
        let ecosystem = ecosystems[i];
        this.enabledEcosystems.push(ecosystem.name);
      }
      this.vulnsourceEnabled = this.enabledEcosystems.length !== 0;
    },
    saveUrl: function () {
      /*
      this.updateConfigProperties([
        {
          groupName: 'vuln-source',
          propertyName: 'google.osv.base.url',
          propertyValue: this.osvBaseUrl,
        },
        {
          groupName: 'vuln-source',
          propertyName: 'google.osv.enabled',
          propertyValue: this.enabledEcosystems.join(';'),
        },
        {
          groupName: 'vuln-source',
          propertyName: 'google.osv.alias.sync.enabled',
          propertyValue: this.aliasSyncEnabled,
        },
      ]);*/
    },
  },
  created() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'vuln-source';
      });
      /*for (let i = 0; i < configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case 'google.osv.enabled':
            this.ecosystemConfig = item.propertyValue;
            this.vulnsourceEnabled = this.ecosystemConfig != null;
            break;
          case 'google.osv.alias.sync.enabled':
            this.aliasSyncEnabled = common.toBoolean(item.propertyValue);
            break;
          case 'google.osv.base.url':
            this.osvBaseUrl = item.propertyValue;
            break;
        }
      }*/
      this.enabledEcosystems = this.ecosystemConfig
        .split(';')
        .map((ecosystem) => ecosystem.trim());
    });
  },
};
</script>
