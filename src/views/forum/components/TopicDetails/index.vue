<template>
  <div class="app-container">
    <el-row :gutter="14">
      <el-col class="main" :span="15" :offset="3">
        <div class="panel">
          <el-card class="box-card" shadow="never">
            <div slot="header" class="clearfix">
              <span class="topic-full-title">
                <el-tag v-if="topic_details.topicType === 'share'" size="small">分享</el-tag>
                <el-tag v-else-if="topic_details.topicType === 'ask'" size="small" type="success">问答</el-tag>
                <span class="title">{{ topic_details.title }}</span>
                <div class="title-details">
                  <span>发布于 {{ topic_details.create_at | formatLocalTime(topic_details.create_at) }}</span>
                  <span>作者<a :href="'#' + topic_details.author.author_name">{{ topic_details.author.author_name }}</a></span>
                  <span>{{ topic_details.pageviews }} 次浏览</span>
                  <span v-if="topic_details.topicType === 'share'">来自 分享</span>
                  <span v-else-if="topic_details.topicType === 'ask'">来自 问答</span>
                  <div class="handle">
                    <router-link :to="'/forum/edit/'+topic_details.id"><el-button type="primary" icon="el-icon-edit" class="edit" circle /></router-link>
                    <el-button type="warning" icon="el-icon-star-off" class="bookmark" circle />
                  </div>
                </div>
              </span>
            </div>
            <div class="main-content">
              {{ topic_details.content }}
            </div>
          </el-card><br>
          <el-card v-if="topic_details.reply_count !== 0" class="box-card" shadow="never">
            <div slot="header" class="clearfix">
              <span>{{ topic_details.reply_count }} 回复</span>
            </div>
            <div v-for="(item, index) in topic_details.replies" :key="item.id" class="reply-list">
              <div :id="item.id" class="reply-item">
                <div class="author-content">
                  <a :href="'#' + item.author.author_name" class="user-avatar">
                    <img :src="item.author.avatar_url">
                  </a>
                  <div class="user-info">
                    <a class="dark reply-author" :href="'#' + item.author.author_name">{{ item.author.author_name }}</a>
                    <a class="reply-time" :href="'#' + item.id">{{ index + 1 }}楼 • {{ item.create_at | formatLocalTime(item.create_at) }}</a>
                  </div>
                  <div class="user-action">
                    <span>
                      <svg-icon icon-class="reply" title="回复" @click="showReply(index)" />
                    </span>
                  </div>
                </div>
                <div class="reply-content">
                  {{ item.content }}
                </div>
                <div class="clearfix">
                  <div v-show="isShow.includes(index)" class="reply2-area">
                    <el-form ref="replyForm" :model="replyForm">
                      <el-form-item>
                        <markdown-editor v-model="replyForm.content" height="200px" />
                      </el-form-item>
                      <el-form-item>
                        <el-button type="primary" @click="onSubmit('replyForm')">回复</el-button>
                        <el-button @click="handleCancel(index)">取消</el-button>
                      </el-form-item>
                    </el-form>
                  </div>
                </div>
              </div>
            </div>
          </el-card><br>
          <el-card class="box-card" shadow="never">
            <div slot="header" class="clearfix">
              <span>添加回复</span>
            </div>
            <el-form ref="addReplyForm" :model="addReplyForm">
              <el-form-item>
                <markdown-editor v-model="addReplyForm.content" height="200px" />
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="onSubmit('addReplyForm')">回复</el-button>
              </el-form-item>
            </el-form>
          </el-card>
        </div>
      </el-col>
      <el-col :span="4">
        <el-card class="box-card" shadow="never">
          <div slot="header" class="clearfix">
            <span>作者</span>
          </div>
          <div class="card-main">
            <a :href="'https:\/\/cnodejs.org\/user\/' + topic_details.author.loginname" class="user-avatar" target="_blank">
              <img :src="topic_details.author.avatar_url" :title="topic_details.author.loginname">
            </a>
            <span class="user-name">
              <a class="dark" :href="'#' + topic_details.author.author_name">{{ topic_details.author.author_name }}</a>
            </span>
            <div class="user-details">
              预留部分  包括上传 下载  分享率 等...
            </div>
          </div>
        </el-card><br>
        <el-card class="box-card" shadow="never">
          <div slot="header" class="clearfix">
            <span>该部分待考虑</span>
          </div>
          <div v-for="item in author_topics.recent_topics" :key="item">
            <a class="topic-title" :href="'https:\/\/cnodejs.org\/topic\/' + item.id">{{ item.title }}</a>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import MarkdownEditor from '@/components/MarkdownEditor'
import { fetchDetails } from '@/api/forum'
import { formatTime, timeStamp } from '@/utils'
export default {
  components: {
    MarkdownEditor
  },
  filters: {
    formatLocalTime(time) {
      return formatTime(timeStamp(time))
    }
  },
  data() {
    return {
      topic_details: [],
      author_topics: [],
      isShow: [],
      replyForm: { // 回复其他回复该话题的操作
        content: ''
      },
      addReplyForm: { // 自己回复这个话题的操作
        content: ''
      }
    }
  },
  created() {
    const id = this.$route.params && this.$route.params.id
    this.getDetails(id)
  },
  methods: {
    getDetails(id) {
      fetchDetails(id).then(res => {
        this.topic_details = res.data
      })
    },
    showReply(id) {
      if (this.isShow.indexOf(id) !== -1) {
        return false
      } else {
        this.isShow.push(id)
      }
    },
    handleCancel(id) {
      const index = this.isShow.indexOf(id)
      this.isShow.splice(index, 1)
    },
    onSubmit(formName) {
      console.log(this.$refs[formName].model.content)
    }
  }
}
</script>

<style lang="scss" scoped>
.clearfix {
  display: block;
  .topic-full-title {
    .title {
      font-size: 24px;
    }
  }
  .title-details {
    display: inline-block;
    width: 100%;
    font-size: 12px;
    color: #838383;
    line-height: 40px;
    span::before {
      content: "*";
    }
    .handle {
      float: right;
    }
  }
}
.main-content {
  display: block;
  margin: 0 10px;
  text-indent: 2rem;
}
.reply-item {
  .author-content {
    // display: inline-flex;
    display: block;
    .user-avatar {
      display: inline-block;
      img {
        width: 30px;
        height: 30px;
      }
    }
    .user-info {
      display: inline-block;
      margin-left: 10px;
      .reply-author {
        font-size: 12px;
        font-weight: 700;
        color: #666;
      }
      .reply-author + a:hover {
        text-decoration: underline;
      }
      .reply-time {
        font-size: 11px;
        color: #08c;
      }
    }
    .user-action {
      color: #666;
      float: right;
    }
  }
  .reply-content {
    display: block;
    margin: 10px 0;
    text-indent: 2rem;
  }
}
// .markdown-text {
//   p {
//     a {
//       color: #08c;
//     }
//     a:hover {
//       text-decoration: underline;
//     }
//   }
// }

.card-main {
  display: block;
  text-align:center;
  .user-avatar {
    vertical-align: middle;
    img {
      width: 48px;
      height: 48px;
    }
  }
  .user-name {
    font-size: 16px;
    max-width: 120px;
    white-space: nowrap;
    .dark {
      color: #778087;
    }
  }
}
.topic-title {
  overflow: hidden;
  width: 90%;
  color: #778087;
  white-space: nowrap;
  display: inline-block;
  vertical-align: middle;
  font-size: 16px;
  line-height: 20px;
  text-overflow: ellipsis;
}
</style>
