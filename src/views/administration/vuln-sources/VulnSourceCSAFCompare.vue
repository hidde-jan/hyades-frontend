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
        :old-string="JSON.stringify(leftContent,null,2)"
        :new-string="JSON.stringify(rightContent,null,2)"
        output-format="side-by-side"
        :filename="leftTitle"
        :newFilename="rightTitle"
        language="plaintext"
        :hideStat=false
        theme="dark"
        :forceInlineComparison=false
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
    leftTitle: String,
    leftContent: Object,
    rightTitle: String,
    rightContent: Object,
  },
  components: {
    CodeDiff
  },
};
</script>
