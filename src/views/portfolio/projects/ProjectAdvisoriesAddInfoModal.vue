<template>
  <b-modal
    id="projectAdvisoriesAddInfoModal"
    size="md"
    hide-header-close
    no-stacking
    :title="modalTitle"
  >
    <div class="p-4">
      <div class="form-group">
        <label for="statusSelect">Status</label>
        <select v-model="selectedStatus" id="statusSelect" class="form-control">
          <option value="" disabled>Select a status</option>
          <option value="not remediated">not remediated</option>
          <option value="remediation in progress">
            remediation in progress
          </option>
          <option value="remediation done">remediation done</option>
        </select>
      </div>
      <div class="form-group">
        <label for="commentTextarea">Comment</label>
        <textarea
          v-model="comment"
          id="commentTextarea"
          class="form-control"
          rows="4"
          placeholder="Enter your comment here"
        ></textarea>
      </div>
    </div>
    <hr />
    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
        $t('message.close')
      }}</b-button>
      <b-button size="md" variant="primary" @click="save()">{{
        $t('admin.save')
      }}</b-button>
    </template>
  </b-modal>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import BInputGroupFormSelect from '../../../forms/BInputGroupFormSelect';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';

export default {
  name: 'projectAdvisoriesAddInfoModal',
  props: {
    ids: {
      type: Array,
    },
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
      selectedStatus: '',
      comment: '',
      modalTitle: 'Add audit information',
    };
  },
  methods: {
    save() {
      const url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_AGGREGATOR}`;  // TODO: Update URL
      this.ids.forEach((element, index) => {
        this.axios
          .put(url, {
            projectId: element,
            status: this.selectedStatus,
            comment: this.comment,
          })
          .then((response) => {
            this.$toastr.s(this.$t('message.updated'));
          })
          .catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
      });
      this.$bvModal.hide('projectAdvisoriesAddInfoModal');
    },
    close() {
      this.$emit('close');
    },
  },
};
</script>
