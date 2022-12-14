<template>
  <main class="main-view">
    <NavbarLeft />
    <LoadingSpinner v-if="isProcessing"/>
    <template v-else>
      <div class="tweet_list">
        <div class="tweet_list-title d-flex">
          <a @click="$router.back()"><i class="fa-solid fa-arrow-left size-20"></i></a><h4>推文</h4>
        </div>
        <div class="scroll">
          <div class="comment_wrap detail d-flex" v-if="tweet.User">
            <div class="comment_wrap_body">
              <div class="d-flex">
                <div class="avatar_image"><router-link :to="{name: 'user-page', params: {id: tweet.UserId}}"><img :src="tweet.User.avatar | emptyImage" /></router-link></div>
                <div class="comment_wrap_body--title">
                  <h5 class="size-16"><router-link :to="{name: 'user-page', params: {id: tweet.UserId}}">{{tweet.User.name}}</router-link></h5>
                  <p class="size-14"><router-link :to="{name: 'user-page', params: {id: tweet.UserId}}">{{tweet.User.account | account}}</router-link></p>
                </div>
              </div>
              <div class="comment_wrap_body--content size-24 mb-2">{{tweet.description}}</div>
              <div class="comment_wrap_body--time">
                <span class="size-14">{{tweet.createdAt | getTime }}</span>
                ・
                <span class="size-14">{{tweet.createdAt | fullDate }}</span>
              </div>
            </div>
            <div class="comment_wrap_footer d-flex mt-3 detail">
                <div class="comment_wrap_footer--comments-num d-flex mr-10">
                  <div class="number-wrap mr-1">{{tweet.repliedCount}}</div><p>回覆</p>
                </div>
                <div class="comment_wrap_footer--liked-num d-flex">
                  <div class="number-wrap mr-1">{{tweet.likedCount}}</div><p>喜歡次數</p>
                </div>
              </div>
          </div> 
          <div class="tweet_list-detail btn-wrap d-flex">
            <div class="icon-btn reply" @click.stop.prevent="openModal"></div>
            <div class="icon-btn liked isliked my-auto" v-if="tweet.isFav" @click.stop.prevent="deleteLike(tweet.id)"></div>
            <div class="icon-btn liked my-auto" v-else @click.stop.prevent="addLike(tweet.id)"></div>
          </div>
          <ReplyTweets :reply-to="tweet" />
        </div>
      </div>
    <SuggestUser />
    <TweetReply :is-show="isShow" :tweet = "tweet" @close-modal="closeModal" @refresh-replies="refreshAgain"/>
    </template>
  </main>
</template>
<script>
import NavbarLeft from "../components/NavbarLeft"
import SuggestUser from "../components/SuggestUser"
import ReplyTweets from "../components/ReplyTweets"
import TweetReply from "../components/TweetReply"
import tweetsAPI from '../apis/tweets'
import LoadingSpinner from '../components/LoadingSpinner'
import { accountFilter } from './../utils/mixins'
import { fromNowFilter } from './../utils/mixins'
import { emptyImageFilter } from './../utils/mixins'
import { Toast } from "@/utils/helpers"
import { mapState } from 'vuex'

export default{
  data () {
    return{
      tweet: {},
      isShow: false,
      isProcessing: false
    }
  },
  components: {
    NavbarLeft,
    SuggestUser,
    ReplyTweets,
    TweetReply,
    LoadingSpinner
  },
  mixins: [accountFilter, emptyImageFilter,fromNowFilter],
  methods: {
    async addLike (tweetId) {
      try{
        const data = tweetsAPI.addLike({ tweetId })
        if(data.status === "error"){
          throw new Error(data.message)
        }
        if(this.tweet.id === tweetId){
          this.tweet.isFav = true
          this.tweet.likedCount = this.tweet.likedCount + 1
        }
      }catch(err){
        console.error(err)
        Toast.fire({
          icon: 'error',
          title: '無法加入最愛，請稍後再試'
        })
      }
    },
    async deleteLike (tweetId) {
      try{
        const data = tweetsAPI.deleteLike({ tweetId })
        if(data.status === "error"){
          throw new Error(data.message)
        }
        if(this.tweet.id === tweetId){
          this.tweet.isFav = false
          this.tweet.likedCount = this.tweet.likedCount - 1
        }
      }catch(err){
        console.error(err)
        Toast.fire({
          icon: 'error',
          title: '無法刪除最愛，請稍後再試'
        })
      }
    },
    openModal (){
      return this.isShow = true
    },
    closeModal (show){
      return this.isShow = show
    },
    async fetchTweet (tweetId) {
      try {
        this.isProcessing = true
        const { data } = await tweetsAPI.getTweet({ tweetId })
        if (data.status === 'error') {
          throw new Error(data.message)
        }
        this.tweet = this.getLikedStatus(data)
        this.isProcessing = false
      }catch(err){
        this.isProcessing = false
        console.error(err)
        Toast.fire({
          icon: 'error',
          title: '無法找到推文，請稍後再試'
        })
      }
    },
    getLikedStatus (data) {
      const result = data.Likes.filter(obj => obj.UserId === JSON.parse(this.currentUser.id))       
      if(result.length > 0) {
        data = {
          ...data,
          isFav: true
        }
      }else{
        data = {
          ...data,
          isFav: false
        }
      }
      return this.tweet = data
    },
    refreshAgain (tweetId){
      this.fetchTweet(tweetId)
    }
  },
  computed: {
    ...mapState(['currentUser'])
  },
  beforeRouteUpdate (to, from, next) {
    // 路由改變時重新抓取資料
    const { id } = to.params
    this.fetchTweet(id)
    next()
  },
  created () {
    const { id: tweetId } = this.$route.params
    this.fetchTweet(tweetId)
  }
}
</script>

<style lang="scss" scoped>
@import '../assets/scss/main.scss';
@import '../assets/scss/tweet.scss';
</style>
