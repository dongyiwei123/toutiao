<template>
  <div class="home">
    <header class="header">
      <div class="loge">
        <span class="iconfont iconnew"></span>
      </div>
      <div class="search" @click="$router.push('/search')">
        <span class="iconfont iconsearch"></span>
        <span>搜索新闻</span>
      </div>
      <div class="user" @click="$router.push('/User')">
        <span class="iconfont iconwode"></span>
      </div>
    </header>
    <!-- tab栏 -->
    <van-sticky>
      <span class="iconfont iconicon-test" @click="$router.push('/category')"></span>
    </van-sticky>
    <nav class="category">
      <van-tabs v-model="active" sticky animated swipeable>
        <van-tab :title="item.name" v-for="item in categoryList" :key="item.id">
          <van-pull-refresh v-model="isLoading" @refresh="onRefresh">
            <van-list
              v-model="loading"
              :finished="finished"
              finished-text="没有更多了"
              @load="onLoad"
              :immediate-check="false"
            >
              <myNew :post="item" @click="goDetail" v-for="item in postList" :key="item.id"></myNew>
            </van-list>
          </van-pull-refresh>
        </van-tab>
      </van-tabs>
    </nav>
  </div>
</template>

<script>
export default {
  name: 'home',
  data() {
    return {
      active: 0,
      categoryList: [],
      pageIndex: 1,
      pageSize: 5,
      postList: [],
      loading: false,
      finished: false,
      isLoading: false
    }
  },
  created() {
    this.getCategoryList()
  },
  // activated() {
  //   this.getCategoryList()
  // },
  methods: {
    async getCategoryList() {
      const active = JSON.parse(localStorage.getItem('active'))
      if (active) {
        this.categoryList = active
        this.getPostList(this.categoryList[0].id)
        return
      }
      const { data: res } = await this.$axios.get('/category')
      const { statusCode, data } = res
      if (statusCode === 200) {
        this.categoryList = data
        this.getPostList(this.categoryList[0].id)
      }
    },
    async getPostList(id) {
      const { data: res } = await this.$axios.get('/post', {
        params: {
          pageIndex: this.pageIndex,
          pageSize: this.pageSize,
          category: id
        }
      })
      // console.log(res)
      const { statusCode, data } = res
      if (statusCode === 200) {
        if (this.pageIndex === 1) {
          this.postList = []
        }
        this.postList = [...this.postList, ...data]
        this.loading = false
        this.isLoading = false
        if (this.pageSize > data) {
          this.finished = true
        }
      }
    },
    onLoad() {
      // console.log(this.categoryList[0].id)
      this.pageIndex++
      this.getPostList(this.categoryList[this.active].id)
    },
    onRefresh() {
      this.postList = []
      this.pageIndex = 1
      this.loading = true
      this.finished = false
      this.getPostList(this.categoryList[this.active].id)
    },
    goDetail(id) {
      this.$router.push(`/postDetail/${id}`)
    }
  },
  watch: {
    active(value) {
      this.postList = []
      this.pageIndex = 1
      this.finished = false
      this.loading = true
      const id = this.categoryList[value].id
      this.getPostList(id)
    }
  },
  beforeRouteLeave(to, from, next) {
    if (to.name !== 'postDetail') {
      this.$store.commit('del', { name: 'home' })
    }
    next()
  }
}
</script>

<style lang="less" scoped>
.home {
  .header {
    display: flex;
    // align-items: center;
    line-height: 50px;
    height: 50px;
    padding: 0 10px;
    background-color: #f00;
    text-align: center;
    color: #fff;
    .loge,
    .user {
      width: 50px;
    }
    .loge {
      span {
        font-size: 50px;
      }
    }
    .user {
      span {
        font-size: 20px;
      }
    }
    .search {
      flex: 1;
      margin: 8px 15px;
      line-height: 34px;
      background-color: rgba(255, 255, 255, 0.4);
      border-radius: 17px;
      span {
        &:nth-child(1) {
          font-size: 18px;
        }
        margin-right: 5px;
        font-size: 14px;
      }
    }
  }
  /deep/ .van-tabs .van-sticky {
    width: 90%;
  }
  .category {
    position: relative;
    /deep/ .van-tabs__line {
      background-color: pink;
    }
  }
  .iconicon-test {
    position: absolute;
    width: 10%;
    height: 44px;
    right: 0;
    text-align: center;
    line-height: 44px;
    background-color: #fff;
    z-index: 999;
  }
}
</style>
