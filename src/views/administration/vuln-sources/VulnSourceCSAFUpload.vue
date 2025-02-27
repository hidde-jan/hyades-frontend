<template>
  <b-modal
    id="vulnSourceCSAFUpload"
    size="md"
    hide-header-close
    no-stacking
    :title="$t('admin.upload_file')"
  >

    <div style="text-align: center;">
      <input type="file" ref="fileInput" @change="onFileChange" style="display: none;" />
      <b-validated-input-group-form-input
        id="fileName"
        :label="$t('admin.file_name')"
      input-group-size="mb-3"
      :value="fileName || $t('admin.no_file_selected')"
      readonly
      style="text-align: center;"
      />
      <b-button size="md" variant="primary" @click="selectFile">
        {{$t('admin.select_file')}}
      </b-button>
    </div>

    <hr>

    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
          $t('message.close')
        }}</b-button>
      <b-button size="md" variant="success" @click="upload(cancel)" class="ml-2">{{
          $t('admin.upload')
        }}</b-button>
    </template>
  </b-modal>
</template>

<script>
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
export default {
  name: 'vulnSourceCSAFUpload',
  props: {
  },
  components: {
    BValidatedInputGroupFormInput,
  },
  data() {
    return {
      selectedFile: null,
      fileName: '',
    };
  },
  methods: {
    selectFile() {
      this.$refs.fileInput.click();
    },
    onFileChange(event) {
      this.selectedFile = event.target.files[0];
      this.fileName = this.selectedFile ? this.selectedFile.name : '';
    },
    async upload(cancel) {
      if (this.selectedFile) {
        try {
          const formData = new FormData();
          formData.append('file', this.selectedFile);
          let url = `${this.$api.BASE_URL}/${this.$api.URL_CSAF_DOCUMENT}`;
          const response = await this.axios.post(url, formData, {
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          });
          cancel();
        } catch (error) {
        }
      } else {
        alert(this.$t('admin.please_select_file'));
      }
    },
    resetFile() {
      this.selectedFile = null;
      this.fileName = '';
    },
  },
};
</script>


