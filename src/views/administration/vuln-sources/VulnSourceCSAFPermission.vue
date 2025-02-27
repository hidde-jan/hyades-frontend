<template>
  <b-modal
    id="vulnSourceCSAFPermission"
    size="xl"
    hide-header-close
    no-stacking
    :title="modalTitle"
    class="custom-modal"
  >
    <div class="modal-body">
      <p>Your account lacks the CSAF_MANAGEMENT permission. The CSAF modules will not function properly until this permission is added.</p>
      <p>You can add it under Administration -> Access Management -> Managed Users -> Permissions or ask your Administrator</p>
    </div>
    <template #modal-footer="{ ok }">
      <b-button @click="ok()">
        OK
      </b-button>
    </template>
  </b-modal>
</template>

<script>
export default {
  name: 'VulnSourceCSAFPermission',
  data() {
    return {
      modalTitle: 'Missing CSAF Permission',
    };
  },
  methods: {
  },
};
</script>
