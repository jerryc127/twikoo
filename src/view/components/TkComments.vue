<template>
  <div class="tk-comments">
    <tk-submit @load="initComments" :config="config" />
    <div class="tk-comments-container" v-loading="loading">
      <div class="tk-comments-title">
        <span class="tk-comments-count">
          <span>{{ count }}</span>
          <span>{{ t('COMMENTS_COUNT_SUFFIX') }}</span>
        </span>
        <span class="tk-icon" v-if="showAdminEntry" v-html="iconSetting" @click="openAdmin"></span>
      </div>
      <div class="tk-comments-no" v-if="!loading && !comments.length">{{ errorMessage }}</div>
      <tk-comment v-for="comment in comments"
        :key="comment.id"
        :comment="comment"
        :replying="replyId === comment.id"
        :config="config"
        @reply="onReply"
        @load="initComments" />
      <div class="tk-expand" v-if="showExpand" @click="onExpand" v-loading="loadingMore">{{ t('COMMENTS_EXPAND') }}</div>
    </div>
  </div>
</template>

<script>
import { call, t } from '../../js/utils'
import TkSubmit from './TkSubmit.vue'
import TkComment from './TkComment.vue'
import iconSetting from '@fortawesome/fontawesome-free/svgs/solid/cog.svg'

export default {
  components: {
    TkSubmit,
    TkComment
  },
  props: {
    showAdminEntry: Boolean
  },
  data () {
    return {
      loading: true,
      loadingMore: false,
      errorMessage: t('COMMENTS_NO_COMMENTS'),
      config: {},
      comments: [],
      showExpand: true,
      count: 0,
      replyId: '',
      iconSetting
    }
  },
  methods: {
    t,
    async initConfig () {
      const result = await call(this.$tcb, 'GET_CONFIG', event)
      if (result && result.result && result.result.config) {
        this.config = result.result.config
      }
    },
    async initComments () {
      this.loading = true
      const url = this.$twikoo.path
        // eslint-disable-next-line no-eval
        ? eval(this.$twikoo.path)
        : window.location.pathname
      await this.getComments({ url })
      this.loading = false
    },
    async onExpand () {
      if (this.loadingMore) return
      this.loadingMore = true
      const url = this.$twikoo.path
        // eslint-disable-next-line no-eval
        ? eval(this.$twikoo.path)
        : window.location.pathname
      const before = this.comments
        .map((item) => item.created)
        .sort((a, b) => a - b)[0] // 最小值
      await this.getComments({ url, before })
      this.loadingMore = false
    },
    onCommentLoaded () {
      typeof this.$twikoo.onCommentLoaded === 'function' && this.$twikoo.onCommentLoaded()
    },
    async getComments (event) {
      try {
        const comments = await call(this.$tcb, 'COMMENT_GET', event)
        if (comments && comments.result && comments.result.data) {
          this.comments = event.before ? this.comments.concat(comments.result.data) : comments.result.data
          this.showExpand = comments.result.more
          this.count = comments.result.count || this.comments.length || 0
          this.$nextTick(this.onCommentLoaded)
        }
      } catch (e) {
        this.errorMessage = e.message
      }
    },
    onReply (id) {
      this.replyId = id
    },
    openAdmin () {
      this.$emit('admin')
    }
  },
  mounted () {
    this.initConfig()
    this.initComments()
  }
}
</script>

<style scoped>
.tk-comments-title {
  font-size: 1.25rem;
  font-weight: bold;
  margin-bottom: 1rem;
  display: flex;
  align-items: baseline;
  justify-content: space-between;
}
.tk-comments-container {
  min-height: 10rem;
  display: flex;
  flex-direction: column;
}
.tk-comments-no {
  flex: 1;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
}
.tk-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  vertical-align: sub;
  height: 0.75em;
  width: 0.75em;
  line-height: 0;
  cursor: pointer;
}
.tk-icon /deep/ svg {
  width: 100%;
  height: 100%;
  fill: #409eff;
}
</style>
