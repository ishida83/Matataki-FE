<template>
  <div class="app">
    <nuxt />
    <!-- <lazy-component> -->
    <g-footer v-if="hideFooter" />
    <!-- </lazy-component> -->
    <back-to-top
      v-if="!hideBackTop"
      :visibility-height="300"
      :back-position="50"
      class="backtop"
      transition-name="fade"
    >
      <svg-icon
        class="backtop-icon"
        icon-class="backtop"
      />
    </back-to-top>
    <feedback
      v-if="!hideFeedback"
      :show-position="100"
    />
    <AuthModal v-model="loginModalShow" />
    <articleImport
      v-model="importModalShow"
      @importArticle="importArticle"
    />
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import AuthModal from '@/components/Auth/index.vue'
import BackToTop from '@/components/BackToTop'
import articleImport from '@/components/article_import/index.vue'
import feedback from '@/components/feedback'
import footer from '~/components/footer/index.vue'
export default {
  name: 'Default',
  components: {
    gFooter: footer,
    AuthModal,
    BackToTop,
    articleImport,
    feedback
  },
  data() {
    return {
      time: null
    }
  },
  computed: {
    ...mapGetters(['isLogined']),
    loginModalShow: {
      get() {
        return this.$store.state.loginModalShow
      },
      set(v) {
        if(v && this.isLogined) return

        this.$store.commit('setLoginModal', v)
      }
    },
    importModalShow: {
      get() {
        return this.$store.state.importArticle.importModalShow
      },
      set(v) {
        this.$store.commit('importArticle/setImportModal', v)
      }
    },
    hideBackTop() {
      return this.$route.name === 'publish-type-id'
    },
    hideFooter() {
      const hide = ['publish-type-id', 'login-github', 'index']
      return !hide.includes(this.$route.name)
    },
    hideFeedback() {
      return this.$route.name === 'publish-type-id'
    }
  },
  watch: {
    isLogined(val) {
      if(val) this.loginModalShow = false
    }
  },
  created() {
    if(this.$route.query.referral && !this.isLogined) {
      setTimeout(()=> {
        this.loginModalShow = true
      }, 100)
    }
  },
  mounted() {
    this.$store.dispatch('testLogin')
    this.removeOverflowHide()
  },
  methods: {
    importArticle() {
    },
    removeOverflowHide() {
      // 这段代码也是无奈之举
      // 这里的代码, 如果没有找到为什么会设置 overflow hideen, 就删除了的话, 就等着加班吧 !!!
      clearInterval(this.time)
      this.time = setInterval(() => {
        const bodyDom = document.querySelector('body')
        if (bodyDom.style.overflow) {
          const dialog = document.querySelectorAll('.el-dialog__wrapper')
          let dialogStatus = false
          // 循环 所有元素有一个不是node 说明是展开的dialog
          for (let i = 0; i < dialog.length; i++) {
            if (dialog[i].style.display !== 'none') {
              dialogStatus = true
              break
            }
          }
          // 此时body hidden 否则 auto
          if (dialogStatus) {
            bodyDom.style.overflow = 'hidden'
          } else {
            bodyDom.style.overflow = 'auto'
          }
        }
      }, 1000)
    }
  }
}
</script>

<style lang="less" scoped>
.app .backtop {
  border-radius: 50%;
  width: 50px;
  height: 50px;
  background: @purpleDark;
  cursor: pointer;
  z-index: 99;
  font-size: 18px;
  position: fixed;
  right: 40px;
  bottom: 110px;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  // box-shadow: 0 4px 24px rgba(84, 45, 224, .5);
  &:hover {
    opacity: 0.9;
  }

  &-icon {
    color: #fff;
  }
}

</style>
