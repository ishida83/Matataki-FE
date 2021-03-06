<template>
  <div class="notification">
    <g-header />
    <el-row class="notification-container">
      <el-col v-show="!showDetails" :span="16">
        <div class="notification-topbar">
          <h3 class="notification-topbar-title">
            通知
          </h3>
        </div>
        <notifyCard
          v-for="(item, index) in notifications"
          :key="index"
          :card="item"
          :user="getUser(item.user_id)"
          :post="getPost(item)"
          :comment="getComment(item)"
          @openDetails="openDetails"
        />
        <div v-if="notifications.length === 0" class="noData">
          {{ $t('notContent') }}
        </div>
        <div class="load-more">
          <buttonLoadMore
            :type-index="0"
            :params="pull.params"
            :api-url="pull.apiUrl"
            return-type="Object"
            @buttonLoadMore="buttonLoadMore"
          />
        </div>
      </el-col>
      <el-col v-if="showDetails" :span="16" class="detail">
        <div class="fl detail-topbar">
          <div class="detail-topbar-back" @click="closeDetails">
            <i class="el-icon-arrow-left" />
          </div>
          <h3>
            {{ actionDetailLabels[detailsIndex.action] }}
          </h3>
        </div>
        <div style="margin-bottom: 20px;">
          <objectCard
            v-if="detailsIndex.objectType === 'article'"
            mode="post"
            :post="posts.find(post => post.id === detailsIndex.objectId)"
            bg-color="white"
          />
        </div>
        <el-divider />
        <div v-for="(item, index) in notificationDetails" :key="index" style="margin: 20px 0;">
          <objectCard
            v-if="detailsIndex.action === 'like' || detailsIndex.action === 'follow'"
            mode="user"
            :user="item.user"
            :create-time="item.create_time"
            :action="item.action"
            bg-color="white"
          />
          <commentCard
            v-if="detailsIndex.action === 'comment'"
            :card="item"
          />
        </div>

        <div class="load-more">
          <buttonLoadMore
            :type-index="0"
            :params="detailPull.params"
            :api-url="detailPull.apiUrl"
            return-type="Object"
            @buttonLoadMore="detailLoadMore"
          />
        </div>
      </el-col>
      <el-col :span="8">
        <!-- 消息筛选 -->
        <div class="option">
          <h3 class="option-title">
            消息筛选
          </h3>
          <div class="option-card">
            <!-- <el-checkbox
              v-model="checkAll"
              :indeterminate="isIndeterminate"
              @change="handleCheckAllChange"
            >
              全选
            </el-checkbox> -->
            <el-checkbox-group
              v-model="checkedCities"
              class="fl checkbox-group"
              @change="handleCheckedCitiesChange"
            >
              <el-checkbox
                v-for="city in cities"
                :key="city"
                disabled
                class="checkbox"
                :label="city"
              >
                {{ city }}
              </el-checkbox>
            </el-checkbox-group>
          </div>
        </div>
        <!-- 查看模式 -->
        <div class="option">
          <h3 class="option-title">
            查看模式
          </h3>
          <div class="option-card">
            <el-radio v-model="viewMode" disabled label="all">
              查看全部
            </el-radio>
            <el-radio v-model="viewMode" disabled label="unread">
              只看未读
            </el-radio>
            <el-divider />
            <div class="option-card-button">
              <el-button size="medium" plain @click="notifyMarkReadAll">
                <svg-icon icon-class="read-all" />
                全部标记为已读
              </el-button>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import buttonLoadMore from '@/components/button_load_more/index.vue'
import notifyCard from '@/components/notification/card.vue'
import objectCard from '@/components/notification/objectCard.vue'
import commentCard from '@/components/notification/commentCard.vue'

export default {
  name: 'NotificationPage',
  components: { buttonLoadMore, notifyCard, objectCard, commentCard },
  data() {
    return {
      showDetails: false,
      notifications: [],
      users: [],
      posts: [],
      comments: [],
      detailsIndex: null,
      notificationDetails: [],
      pagePosition: 0,
      actionDetailLabels: {
        like: '推荐详情',
        comment: '评论详情',
        follow: '关注详情'
      },
      checkAll: true,
      checkedCities: [],
      cities: ['评论信息', '推荐信息', '关注信息', '打赏信息', '发文信息', '系统通知'],
      isIndeterminate: true,
      viewMode: 'all',
      startId: 0
    }
  },
  computed: {
    pull() {
      return {
        apiUrl: 'notifyCenter',
        params: { pagesize: 20, startId: this.startId }
      }
    },
    detailPull() {
      return {
        apiUrl: 'notifyDetails',
        params: { pagesize: 20, ...this.detailsIndex }
      }
    }
  },
  created() {
    this.handleCheckAllChange(true)
  },
  methods: {
    buttonLoadMore(res) {
      if(res.data && res.data.list.length > 0 ) {
        this.notifications.push(...res.data.list)
        this.users.push(...res.data.users)
        this.posts.push(...res.data.posts)
        this.comments.push(...res.data.comments)
        // 标记已读
        this.markRead(res.data.list)
        // 设定起始查询位置
        if(!this.startId) this.startId = res.data.list[0].id
      }
    },
    detailLoadMore(res) {
      if(res.data && res.data.list.length > 0 ) {
        this.notificationDetails.push(...res.data.list)
      }
    },
    getUser(userId) {
      if(this.users && this.users.length > 0)
        return this.users.find(user => user.id === userId)
      return null
    },
    getPost(notify) {
      if(this.posts && this.posts.length > 0 && notify.object_type === 'article') {
        const postId = notify.object_id
        return this.posts.find(post => post.id === postId)
      }
      return null
    },
    getComment(notify) {
      if(this.comments && this.comments.length > 0 && notify.action === 'comment') {
        const commentId = notify.remark
        return this.comments.find(comment => comment.id === commentId)
      }
      return null
    },
    openDetails(data) {
      this.pagePosition = document.documentElement.scrollTop
      window.scroll(0, 0)
      this.detailsIndex = data
      this.showDetails = true
    },
    closeDetails() {
      this.showDetails = false
      this.notificationDetails = []
      setTimeout(() => window.scroll(0, this.pagePosition), 100)
    },
    /** 标记已读 */
    markRead(notifications) {
      let notifyIds = []
      notifications.forEach(item => {
        if(item.state === 0) notifyIds.push(item.id)
      })
      if(notifyIds.length > 0) this.$API.notifyMarkRead(notifyIds)
    },
    handleCheckAllChange(val) {
      this.checkedCities = val ? this.cities : []
      this.isIndeterminate = false
    },
    handleCheckedCitiesChange(value) {
      console.log(value)
      let checkedCount = value.length
      this.checkAll = checkedCount === this.cities.length
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.cities.length
    },
    async notifyMarkReadAll() {
      const res = await this.$API.notifyMarkReadAll()
      if(res.code === 0) {
        this.$message.success(this.$t('success.success'))
        this.$router.go(0)
      }
      else this.$message.error(this.$t('error.fail'))
    },
  }
}
</script>
<style lang="less" scoped src="./index.less"></style>
<style lang="less" scoped>
.load-more /deep/ button.btn {
  margin: 50px auto !important;
  width: 300px !important;
  height: 40px !important;
  font-size: 16px !important;
}
.notification-container /deep/ {
  .el-col-8 {
    padding: 0 10px;
  }
  .el-col-16 {
    padding: 0 10px;
  }
}
// 小于768
@media screen and (max-width: 768px){
  .notification-container /deep/ {
    .el-col-8 {
      width: 100%;
      margin-bottom: 10px;

      // 先隐藏一下
      display:block;
      overflow: hidden;
      width: 0;
      height: 0;
    }
    .el-col-16 {
      width: 100%;
    }
  }
}
</style>
