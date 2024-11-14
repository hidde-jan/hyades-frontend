<template>
  <b-modal
    id="vulnSourceCSAFCompareModal"
    size="xl"
    hide-header-close
    no-stacking
    :title="$t('admin.compare_selected')"
  >

    <div>
      <CodeDiff
        :old-string="leftContent"
        :new-string="rightContent"
        output-format="side-by-side"
        :filename="leftTitle"
        :newFilename="rightTitle"
        language="plaintext"
        :hideStat=false
        theme="dark"
      />
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
import { CodeDiff } from 'v-code-diff'
export default {
  name: 'vulnSourceCSAFCompare',
  props: {
    type: String,
    leftTitle: String,
    leftContent: String,
    rightTitle: String,
    rightContent: String,
  },
  components: {
    CodeDiff
  },
};
</script>
