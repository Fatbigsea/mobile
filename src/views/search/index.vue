<template>
  <div class="search-page">
    <!-- 搜索 -->
    <form action="/" class="search-from">
      <van-search
        v-model="value"
        placeholder="请输入搜索关键词"
        show-action
        @search="onSearch"
        @cancel="onCancel"
        @input="onSug"
        @focus="isResultShow=false"
      />
    </form>

      <!-- 搜索内容列表 -->
        <search-result
          v-if="isResultShow"
          :q="value"
        ></search-result>

       <!-- 搜索推荐 -->
        <van-cell-group v-else-if="value">
          <van-cell
            icon="search"
            v-for="(item,index) in suggestion"
            :key="index"
            @click="onSugClick(item)"
            >
            <div slot="title" v-html="highlight(item)"></div>
          </van-cell>
        </van-cell-group>

             <!-- 浏览历史 -->
        <van-cell-group v-else >
          <van-cell title="历史记录" >
            <template v-if="isDeleteShow">
            <span @click="searchHistory=[]" >全部删除</span>
            &nbsp; &nbsp;
            <span @click="isDeleteShow=false">完成</span>
            </template>
            <van-icon
              slot="right-icon"
              name="delete"
              style="line-height: inherit;"
              @click="isDeleteShow=true"
              v-else
            />
          </van-cell>
          <van-cell
           v-for="(item,index) in searchHistory"
           :key="index"
           icon="search"
           >
           <span @click="onHisClick(item)">{{item}}</span>
            <van-icon
            v-show="isDeleteShow"
            slot="right-icon"
            name="close"
            style="line-height: inherit;"
            @click="searchHistory.splice(index,1)"
            />
          </van-cell>
        </van-cell-group>

  </div>
</template>

<script>
import SearchResult from '@/components/search/search-result'
import { getSuggestion } from '@/api/search'
import { setItem, getItem } from '@/utils/storage'
import { debounce } from 'lodash'
export default {
  name: 'SearchName',
  data () {
    return {
      value: '',
      isResultShow: false,
      suggestion: [],
      isDeleteShow: false,
      searchHistory: getItem('search-history') || []
    }
  },
  components: {
    SearchResult
  },
  watch: {
    searchHistory (newVal) {
      setItem('search-history', newVal)
    }
  },
  methods: {
    onCancel () {
      this.$router.push('/')
    },
    onSearch () {
      const index = this.searchHistory.indexOf(this.value)
      if (index !== -1) {
        this.searchHistory.splice(index, 1)
      }
      this.searchHistory.unshift(this.value)
      this.isResultShow = true
    },
    // 加入函数防抖获取搜索推荐列表
    onSug: debounce(async function () {
      const searchText = this.value
      if (!searchText) {
        return
      }
      const { data } = await getSuggestion(searchText)
      this.suggestion = data.data.options
    }, 400),
    // 获取搜索推荐列表
    // async onSug () {
    //   const searchText = this.value
    //   if (!searchText) {
    //     return
    //   }
    //   const { data } = await getSuggestion(searchText)
    //   this.suggestion = data.data.options
    // },
    // 高亮显示
    highlight (str) {
      return str.toLowerCase().replace(this.value.toLowerCase(),
        `<span style="color:red;">${this.value}</span>`
      )
    },
    // 点击推荐的搜索
    onSugClick (str) {
      // 双向数据绑定，value变化，input监听到
      this.value = str
      this.isResultShow = true
    },
    // 点击历史记录搜索
    onHisClick (str) {
      // if (this.isDeleteShow) {
      //   this.searchHistory.splice(index, 1)
      // } else {
      this.value = str
      this.isResultShow = true
      // }
    }
    // 从后台接口获取历史搜索
    // async getHistory () {
    //   const { data } = await getSearchHistories()
    //   this.searchHistory = data.data.keywords
    //   // console.log(this.searchHistory)
    // }
  }
  // created () {
  //   this.getHistory()
  // }

}
</script>

<style scoped lang="less">
.search-page{
  padding-top:54px;
  .search-from{
    position: fixed;
    z-index:2;
    top: 0;
    left: 0;
    right: 0;
    width: 100%;
  }
}

</style>
