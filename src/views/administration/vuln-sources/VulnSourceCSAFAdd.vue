<template>
  <b-modal
    id="vulnSourceCSAFAddModal"
    @hide="resetValues()"
    size="md"
    hide-header-close
    no-stacking
    :title="$t('admin.add_source')"
  >
    <b-validated-input-group-form-input
      id="url"
      :label="$t('admin.url')"
      input-group-size="mb-3"
      rules="required"
      type="url"
      v-model="url"
    />

  </b-modal>
</template>

<script>

</script>



<style scoped>

</style>
