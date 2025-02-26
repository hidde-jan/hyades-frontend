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
      <p>CSAF_MANAGEMENT permission is missing. The CSAF modules will not function properly until this permission is added.</p>
    </div>
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
