<template>
  <div class="poular-articles">
    <h3 class="pa-head-title">
      {{ $t('home.popularArticles') }}
    </h3>
    <div class=" pa-list">
      <div
        v-for="(item, index) in list"
        :key="index"
        class="pa-list-card"
      >
        <router-link :to="{name: 'p-id', params: { id: item.id } }" target="_blank">
          <div class="cover">
            <img :src="cover(item.cover)" alt="cover">
          </div>
          <h4>{{ item.title }}</h4>
        </router-link>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  props: {
    list: {
      type: Array,
      required: true
    }
  },
  methods: {
    cover(src) {
      return src ? this.$ossProcess(src, { h: 390 }) : ''
    }
  }
}
</script>

<style lang="less">
.poular-articles {
  padding-bottom: 20px;
  .pa-head-title {
    font-size: 20px;
    font-weight: 600;
    color: black;
    margin: 0 0 20px;
  }
  .pa-list {
    min-height: 200px;
    border-radius: 10px;
    background-color: white;
    padding: 20px;
    &-card {
      transition: all 0.1s ease-in 0.1s;
      margin-bottom: 20px;
      &:nth-last-child(1) {
        margin-bottom: 0;
      }

      &:hover {
        transform: scale(1.05);
      }
      a {
        display: flex;
        align-items: center;
      }
      .cover {
        width: 80px;
        height: 40px;
        min-width: 80px;
        border-radius: 6px;
        margin-right: 18px;
        overflow: hidden;
        display: flex;
        justify-content: center;
        align-items: center;
        img {
          width: 80px;
          height: 40px;
        }
      }
      h4 {
        font-size: 16px;
        font-weight: 600;
        color: black;
        line-height: 20px;
        margin: 0;
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 2;
        overflow: hidden;
      }
    }
  }
}
</style>
