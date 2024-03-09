<template>
  <b-container>
    <b-card>
    <b-row>
      <b-col cols="12">
        <b-jumbotron header="Fix Stardew Valley Save File" lead="修正星露谷存檔">
          <b-alert show>此程式僅在本地端執行，所有資訊將不會透過網路傳輸。</b-alert>
          <b-alert show>部分記憶功能可能會因為無痕模式、定期清除、手動清除等操作行為而遺失紀錄。</b-alert>
          <b-alert show variant="warning">請先備份原文件!</b-alert>
        </b-jumbotron>
      </b-col>
      <b-col cols="12">
        <b-alert show>檔案一般存放於: %APPDATA%\StardewValley\Saves\{紀錄名稱}_{隨機碼}\{紀錄名稱}_{隨機碼}</b-alert>
        <b-form-file v-model="original_file" :state="Boolean(original_file)" class="my-2" placeholder="請放入損壞紀錄檔案..." drop-placeholder="Drop file here..."></b-form-file>
        <b-form-file v-model="target_file" :state="Boolean(target_file)" class="my-2" placeholder="請放入英文版紀錄檔案..." drop-placeholder="Drop file here..."></b-form-file>
        <b-button variant="primary" href="#" class="my-2" @click="onClickToReplace">確認繼續</b-button>
        <b-button v-show="replaced_content" :href="replaced_xml" :download="original_name" variant="success" class="float-right my-2">下載</b-button>
      </b-col>
    </b-row>
  </b-card>
  </b-container>
</template>

<script>
import LZUTF8 from 'lzutf8';
async function readXMLFileAsString(file) {
  const reader = new FileReader();
  return new Promise((resolve, reject) => {
    reader.onload = e => resolve(e.target.result)
    reader.onerror = e => reject(e.target.error)
    reader.readAsText(file);
  });
}
const BundleDataRegExp = /<bundleData>(.*?)<\/bundleData>/;
function getStorage() {
  const name = "sdv-o-file";
  return {
    get() {
      return LZUTF8.decompress((localStorage.getItem(name)));
    },
    set(data) {
      return localStorage.setItem(name, LZUTF8.compress((data)));
    }
  }
}

export default {
  name: 'fix-panel',
  data() {
    return {
      original_file: null,
      target_file: null,
      replaced_content: null,
    }
  },
  computed: {
    original_name() {
      if (!this.original_file) {
        return "none"
      }
      return this.original_file.name
    },
    replaced_xml() {
      if (!this.replaced_content) {
        return "#"
      }
      const blob = new Blob([this.replaced_content], { type: 'application/xml' });
      return URL.createObjectURL(blob);
    }
  },
  methods: {
    async onClickToReplace() {
      if (!this.original_file || !this.target_file) {
        return
      }
      const store = getStorage()
      const original_content = await readXMLFileAsString(this.original_file)
      store.set(original_content.substring());
      const target_content = await readXMLFileAsString(this.target_file)
      const match = BundleDataRegExp.exec(target_content)
      if (match) {
        const content = match[1];
        this.replaced_content = original_content.replace(BundleDataRegExp, `<bundleData>${content}</bundleData>`)
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
