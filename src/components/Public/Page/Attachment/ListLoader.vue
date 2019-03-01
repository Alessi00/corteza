<template>
  <div>
    <div v-if="mode === 'list'" class="list">
      <div v-for="a in attachments" :key="a.attachmentID" v-if="a" class="item">
        <a :href="a.download">
          <font-awesome-icon :icon="['fas', 'download']"></font-awesome-icon>
          {{a.name}}
        </a>
        (File size: {{ size(a) }}, uploaded {{ uploadedAt(a) }})
      </div>
    </div>

    <div v-else-if="mode === 'grid'" class="grid">
      <div v-for="a in attachments" :key="a.attachmentID" v-if="a" class="item">
        <a :href="a.download">
          <font-awesome-icon
            :icon="['far', 'file-'+ext(a)]"
            title="Open bookmarks"
          ></font-awesome-icon>
          {{a.name}}
        </a>
        (File size: {{ size(a) }}, uploaded {{ uploadedAt(a) }})
      </div>
    </div>

    <div v-else-if="mode === 'single' || 'gallery'" class="single gallery">
      <div v-for="(a, index) in attachments" :key="a.attachmentID" v-if="a" class="item">
        <div v-if="isImage(a)">
          <img :src="a.previewUrl" @click="openLightbox(index)" />
        </div>
        <div v-else>
          <font-awesome-icon
            :icon="['far', 'file-'+ext(a)]"
            title="Open bookmarks"
          ></font-awesome-icon>
          <a :href="a.download">
            Download
          </a>
        </div>
        {{a.name}}
      </div>
    </div>
  </div>
</template>
<script>
import numeral from 'numeral'
import moment from 'moment'
import Attachment from '@/lib/attachment'

export default {
  props: {
    kind: {
      type: String,
      required: true,
    },

    mode: {
      type: String,
      required: true,
    },

    set: {
      type: Array,
      required: true,
    },
  },

  watch: {
    set: {
      immediate: true,
      handler (set) {
        this.attachments = set.map(a => {
          if (typeof a === 'object') {
            return new Attachment(a)
          } else {
            return null
          }
        })

        set.forEach((a, index) => {
          if (typeof a === 'string') {
            this.$crm.attachmentDetails({ kind: this.kind, attachmentID: a }).then(att => {
              this.attachments.splice(index, 1, new Attachment(att))
            })
          }
        })
      },
    },
  },

  data () {
    return {
      attachments: [],
    }
  },

  methods: {
    size (a) {
      return numeral(a.meta.original.size).format('0b')
    },

    uploadedAt (a) {
      return moment(a.updatedAt || a.createdAt).fromNow()
    },

    openLightbox (index) {
      this.$root.$emit('showAttachmentsModal', index, this.attachments)
    },

    ext (a) {
      const { meta } = a
      switch (meta && meta.original ? meta.original.ext : null) {
        case 'odt':
        case 'doc':
        case 'docx':
          return 'word'
        case 'pdf':
          return 'pdf'
        case 'ppt':
        case 'pptx':
          return 'powerpoint'
        case 'zip':
        case 'rar':
          return 'archive'
        case 'xls':
        case 'xlsx':
        case 'csv':
          return 'excel'
        case 'mov':
        case 'mp3':
        case 'mp4':
          return 'video'
        case 'png':
        case 'jpg':
        case 'jpeg':
          return 'image'
        default: return 'alt'
      }
    },
    isImage (a) {
      return /^image\//.test(a.meta.original.mimetype)
    },
  },
}
</script>
<style lang="scss" scoped>
@import "@/assets/sass/_0.declare.scss";

.fa-download {
  color: $black;
  font-size: 12px;
  margin-right: 2px;
  vertical-align: baseline;
}

.grid {
  .item {
    padding: 10px 0;

    a {
      display: block;
      line-height: 13px;
    }

    .svg-inline--fa {
      color: #000;
      font-size: 30px;
      float: left;
      margin-right: 10px;
    }
  }
}

.single {
  .item {
    .svg-inline--fa {
      font-size: 40px;
    }

    img {
      cursor: pointer;
    }
  }
}
</style>