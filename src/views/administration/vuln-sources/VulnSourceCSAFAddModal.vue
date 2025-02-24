<template>
  <b-modal
    id="vulnSourceCSAFAddModal"
    @hide="resetValues()"
    size="md"
    hide-header-close
    no-stacking
    :title="modalTitle"
  >
    <b-validated-input-group-form-input
      id="name"
      :label="$t('message.name')"
      input-group-size="mb-3"
      rules="required"
      v-model="name"
    />
    <b-validated-input-group-form-input
      id="url"
      :label="$t('admin.url')"
      input-group-size="mb-3"
      rules="required"
      type="url"
      v-model="url"
    />

    <div>
      <c-switch color="primary" v-model="internal" label v-bind="labelIcon" />{{
        $t('admin.internal')
      }}
    </div>

    <div>
      <c-switch color="primary" v-model="enabled" label v-bind="labelIcon" />{{
        $t('admin.enabled')
      }}
    </div>

    <hr />

    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
        $t('message.close')
      }}</b-button>
      <b-button size="md" variant="primary" @click="createCsafSource()">{{
        $t('message.create')
      }}</b-button>
    </template>
  </b-modal>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import BInputGroupFormSelect from '../../../forms/BInputGroupFormSelect';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';

export default {
  name: 'vulnSourceCSAFAddModal',
  props: {
    type: String,
  },
  components: {
    cSwitch,
    BInputGroupFormSelect,
    BValidatedInputGroupFormInput,
  },
  created() {
    this.$root.$on('setModalTitle', (title) => {
      this.modalTitle = title;
    });
  },
  data() {
    return {
      name: null,
      modalTitle: '',
      url: null,
      initialRepositoryType: null,
      internal: false,
      enabled: true,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
    };
  },
  methods: {
    createCsafSource: function () {
      let url ='';
      const title = this.modalTitle;
      if (title === this.$t('admin.add_aggregator')) {
        url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}`;
      } else {
        url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_PROVIDER}`;
      }
      this.axios
        .put(url, {
          name: this.name,
          url: this.url,
          enabled: this.enabled,
        })
        .then((response) => {
          this.$emit('refreshTable');
          console.log('Called refresh table from modal');
          this.$toastr.s(this.$t('admin.repository_created'));
          this.$root.$emit('bv::hide::modal', 'vulnSourceCSAFAddModal');
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
          this.$root.$emit('bv::hide::modal', 'vulnSourceCSAFAddModal');
        });
      this.resetValues();
    },
    resetValues: function () {
      this.name = null;
      this.url = null;
      this.enabled = true;
    },
  },
};
</script>
