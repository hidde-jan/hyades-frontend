<template>
  <b-modal
    id="projectAdvisoriesAddInfoModal"
    :title="'Add audit information'"
    @hide="close"
  >
    <template v-slot:modal-content>
      <div>
        <select v-model="selectedStatus">
          <option value="not remediated">not remediated</option>
          <option value="remediation in progress">
            remediation in progress
          </option>
          <option value="remediation done">remediation done</option>
        </select>
        <textarea
          v-model="comment"
          placeholder="Enter your comment here"
        ></textarea>
        <div class="modal-buttons">
          <button @click="save">Save</button>
          <button @click="close">Close</button>
        </div>
      </div>
    </template>
  </b-modal>
</template>

<script>
export default {
  name: 'projectAdvisoriesAddInfoModal',
  props: {
    ids: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      selectedStatus: '',
      comment: '',
    };
  },
  methods: {
    save() {
      const payload = {
        ids: this.ids,
        status: this.selectedStatus,
        comment: this.comment,
        //TODO: send to api
      };

      console.log('Data to be sent:', payload);
      this.close();
    },
    close() {
      this.$emit('close');
    },
  },
};
</script>
