<template>
  <div class="me-view-body" v-title :data-title="title">
    <el-container class="me-view-container">
      <el-main>
        <div class="me-view-card">
          <h1 class="me-view-title">{{article.title}}</h1>
          <div class="me-view-author">
            <div class="me-view-info">
              <span>作者：{{article.author.nickname}}</span>
              <div class="me-view-meta">
                <span>发布时间：{{article.createDate | format}}</span>
              </div>
            <br>
            </div>
            <el-button
              v-if="this.article.author.id == this.$store.state.id"
              @click="editArticle()"
              style="position: absolute;left: 60%;"
              size="mini"
              round
              icon="el-icon-edit">编辑</el-button>
             <el-button
               v-if="this.article.author.id == this.$store.state.id"
               type="danger"
               style="position: absolute;left: 70%;"
               icon="el-icon-delete"
               @click="deleteArticle()"
               circle>
             </el-button>
          </div>
          <div class="me-view-content">
            <markdown-editor :editor=article.editor></markdown-editor>
          </div>
          <div class="me-view-end">
            <el-alert
              title="...End..."
              type="info"
              :closable="false"
              center="true"
            >
            </el-alert>
          </div>
          <div class="me-view-tag">
            标签：
            <!--<el-tag v-for="t in article.tags" :key="t.id" class="me-view-tag-item" size="mini" type="success">{{t.tagName}}</el-tag>-->
            <el-button @click="tagOrCategory('tag', t.id)" size="mini"  v-for="t in article.tags" :key="t.id" type="primary" round   plain  >{{t.tagName}}</el-button>
          </div>
          <div class="me-view-tag">
            文章分类：
            <!--<span style="font-weight: 600">{{article.category.categoryName}}</span>-->
            <el-button @click="tagOrCategory('category', article.category.id)" size="mini" type="primary"   round   plain>{{article.category.categoryName}}</el-button>
          </div>
        </div>
      </el-main>

    </el-container>
  </div>
</template>

<script>
  import MarkdownEditor from '@/components/markdown/MarkdownEditor'
  import {viewArticle} from '@/api/article'
  import {deleteArticleById} from '@/api/article'


  import default_avatar from '@/assets/img/default_avatar.png'

  export default {
    name: 'BlogView',
    created() {
      this.getArticle()
    },
    watch: {
      '$route': 'getArticle'
    },
    data() {
      return {
        article: {
          id: '',
          title: '',
          commentCounts: 0,
          viewCounts: 0,
          summary: '',
          author: {},
          tags: [],
          category:{},
          createDate: '',
          editor: {
            value: '',
            toolbarsFlag: false,
            subfield: false,
            defaultOpen: 'preview'
          }
        }
      }
    },
    computed: {
      avatar() {
        let avatar = this.$store.state.avatar

        if (avatar.length > 0) {
          return avatar
        }
        return default_avatar
      },
      title() {
        return `${this.article.title} - 文章 - Margo`
      }
    },
    methods: {
      deleteArticle() {
        this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.deleteArticleById()
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });
        });
        },
      tagOrCategory(type, id) {
        this.$router.push({path: `/${type}/${id}`})
      },
      deleteArticleById() {
        let that = this
        deleteArticleById(that.$route.params.id)
        // this.$router.push({path: `/`})
      },
      editArticle() {
        this.$router.push({path: `/write/${this.article.id}`})
      },
      getArticle() {
        let that = this
        viewArticle(that.$route.params.id).then(data => {
          Object.assign(that.article, data.data)
          that.article.editor.value = data.data.body.content

        }).catch(error => {
          if (error !== 'error') {
            that.$message({type: 'error', message: '文章加载失败', showClose: true})
          }
        })
      },

    },
    components: {
      'markdown-editor': MarkdownEditor,

    },
    //组件内的守卫 调整body的背景色
    beforeRouteEnter(to, from, next) {
      window.document.body.style.backgroundColor = '#fff';
      next();
    },
    beforeRouteLeave(to, from, next) {
      window.document.body.style.backgroundColor = '#f5f5f5';
      next();
    }
  }
</script>

<style>
  .me-view-body {
    margin: 100px auto 140px;
  }

  .me-view-container {
    width: 800px;
  }

  .el-main {
    overflow: hidden;
  }

  .me-view-title {
    font-size: 34px;
    font-weight: 800;
    line-height: 1.3;
  }

  .me-view-author {
    /*margin: 30px 0;*/
    margin-top: 30px;
    vertical-align: middle;
  }

  .me-view-picture {
    width: 40px;
    height: 40px;
    border: 1px solid #ddd;
    border-radius: 50%;
    vertical-align: middle;
    background-color: #409EFF;
  }

  .me-view-info {
    display: inline-block;
    vertical-align: middle;
    margin-left: 8px;
  }

  .me-view-meta {
    font-size: 12px;
    color: #969696;
  }

  .me-view-end {
    margin-top: 20px;
  }

  .me-view-tag {
    margin-top: 20px;
    padding-left: 6px;
    border-left: 4px solid #c5cac3;
  }

  .me-view-tag-item {
    margin: 0 4px;
  }

  .me-view-comment {
    margin-top: 60px;
  }

  .me-view-comment-title {
    font-weight: 600;
    border-bottom: 1px solid #f0f0f0;
    padding-bottom: 20px;
  }

  .me-view-comment-write {
    margin-top: 20px;
  }

  .me-view-comment-text {
    font-size: 16px;
  }

  .v-show-content {
    padding: 8px 25px 15px 30px !important;
  }

  .v-note-wrapper .v-note-panel {
    box-shadow: none !important;
  }

  .v-note-wrapper .v-note-panel .v-note-show .v-show-content, .v-note-wrapper .v-note-panel .v-note-show .v-show-content-html {
    background: #fff !important;
  }


</style>
