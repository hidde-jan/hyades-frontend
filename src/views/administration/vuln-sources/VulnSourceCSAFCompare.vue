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
        :old-string="diffOldString"
        :new-string="diffNewString"
        output-format="side-by-side"
        :filename="diffFileName"
        :newFilename="diffNewFileName"
        language="plaintext"
        hideStat=true
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
  },
  components: {
    CodeDiff //Doc: https://github.com/Shimada666/v-code-diff?tab=readme-ov-file#Demo
  },
  created() {
    this.initialRepositoryType = this.type;
    this.repositoryType = this.type;
  },
  data() {
    return {
      diffOldString: "CSAF CSAF CSAF",
      diffNewString: "CASF C$AF CSAF",
      diffFileName: "CSAF 1",
      diffNewFileName: "CSAF 2"
    };
  },
  methods: {
    setFiles(){
      //...
    }
  },
};
</script>


