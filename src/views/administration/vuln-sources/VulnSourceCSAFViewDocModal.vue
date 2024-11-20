<template>
  <b-modal
    id="vulnSourceCSAFViewDocModal"
    size="xl"
    hide-header-close
    no-stacking
    :title="title"
  >

    <div>
      <vue-json-pretty :data="content" />
    </div>

    <hr>

    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
          $t('message.close')
        }}</b-button>
    </template>
  </b-modal>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import 'vue-json-pretty/lib/styles.css';
export default {
  name: 'vulnSourceCSAFViewDocModal',
  props: {
    title: String,
    content: String,
  },
  components: {
    VueJsonPretty,
  },
};
</script>
