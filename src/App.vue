<template>
  <div id="app">
    <template v-if="isWeixn">
      <div class="app">
        <div v-bind:style="{ height: 270 * scale + 'px' }"></div>
        <div class="body">
          <span>活动规则</span>
          <p>1、获得哎哟不错机器人抵用券：500元。</p>
          <p>2、有机会获得价值888元的小机器人，每月由哎哟不错机器人抽取一名幸运粉丝；(中奖者会在公众号公示，请中奖者在一周之内主动联系客服并告知地址以及联系方式以便邮递奖品）。</p>
          <p>3、机器人上市可以第一批优先订购，并享有立减500元的优惠，可与其他优惠券叠加使用。</p>
          <div class="winning_record">
            <span><img src="./assets/img/winning_record.png" @click="winning = true" alt="中奖名单"></span>
          </div>
          <div class="participate">
            <img src="./assets/img/participate_btn_click_before.png" @click="participate = true" alt="参加活动">
          </div>
        </div>
        <!-- 中奖用户 -->
        <transition
          name="winningFade"
          enter-active-class="animated fadeIn"
          leave-active-class="animated fadeOut">
          <div class="winningUser_wrap" v-show="winning">
            <div class="winningUser">
              <img
                v-bind:style="{ width: 600 * scale + 'px', top: 200 * scale + 'px', left: 0, right: 0 }"
                src="./assets/img/winning_record_window.png"
                alt="中奖结果">
              <span
                v-bind:style="{ width: 100 * scale + 'px', height: 100 * scale + 'px', bottom: 100 * scale + 'px', left: 0, right: 0 }"
                @click="winning = false"></span>
            </div>
          </div>
        </transition>
        <!-- 参与手机信息 -->
        <transition
          name="participateFade"
          enter-active-class="animated fadeIn"
          leave-active-class="animated fadeOut">
          <div class="participateFade_wrap" v-show="participate">
            <div class="participateFade">
              <div class="formUser">
                <div class="userData">
                  <label>姓名:</label>
                  <input type="text" v-model="name" autofocus style="width: 3rem;">
                  <span @click="participate = false"></span>
                </div>
                <div class="userData">
                  <label>手机号码:</label>
                  <input type="number" v-model="phone_number">
                </div>
                <div class="userData">
                  <label>地址:</label>
                  <input type="text" v-model="addressBefore" placeholder="省/市">
                </div>
                <div class="detailedAddress">
                  <textarea v-model="addressAfter" placeholder="请输入详细地址!输入有误会影响礼物的发放!"></textarea>
                </div>
                <span class="postData" @click="postData">提交</span>
              </div>
            </div>
          </div>
        </transition>

        <!-- 信息提示 -->
        <transition
          name="participateFade"
          enter-active-class="animated slideInDown"
          leave-active-class="animated slideOutUp">
          <div class="consoleNews" v-show="consoleNews">
            {{ consoleNewsText }}
          </div>
        </transition>
      </div>
    </template>

    <template v-else="isWeixn">
      <div id="errorOpea">请用微信打开!</div>
    </template>
  </div>
</template>

<script>
import '../src/assets/app.less'
export default {
  name: 'app',
  data () {
    return {
      winWidth: window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth, // 页面宽度
      winning: false,
      participate: false,
      consoleNews: false,
      consoleNewsText: '',
      http: 'https://a001.aybc.so/',
      recordTVMFansInfo: 'Shop/recordTVMFansInfo',
      listTVMFansInfo: 'Shop/listTVMFansInfo',
      name: '',
      phone_number: '',
      addressBefore: '',
      addressAfter: '',
      // 判断是不是微信登录
      isWeixn: true
    }
  },

  created: function() {
    // 判断是不是微信打开
    this.is_weixn()
  },
  computed: {
    scale () {
      return this.winWidth / 750
    }
  },
  methods: {
    postData () {
      if (this.name != '') {
        // statement
        if (this.phone_number != '') {
          // statement
          if ((/^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|17[0|1|2|3|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/).test(this.phone_number)) {
            // statement
            if(this.addressBefore != ''){
              // statement
              if (this.addressAfter != '') {
                // statement
                this.$http.post(this.http+this.recordTVMFansInfo,{
                  name: this.name,
                  phone_number: this.phone_number,
                  address: this.addressBefore+this.addressAfter
                },{
                  emulateJSON: true
                })
                .then( (msg) => {
                  if (msg.body.flag == '01') {
                    // statement
                    this.ConsoleNews('参与成功!');
                    setTimeout( (msg) => {
                      this.participate = false;
                      this.name = '';
                      this.phone_number = '';
                      this.addressBefore = '';
                      this.addressAfter = '';
                      window.location = 'https://h5.youzan.com/v2/ump/promocard/fetch?alias=27j9omqm';
                    },2000)
                  } else if (msg.body.flag == '00') {
                    // statement
                    this.ConsoleNews(msg.body.return_code)
                    setTimeout( (msg) => {
                      this.participate = false;
                    },2000)
                  }
                },(response) => {
                  this.ConsoleNews(response.return_code)
                  setTimeout( (msg) => {
                    this.participate = false;
                  },2000)
                })
              } else {
                this.ConsoleNews('请完善详细地址!')
              }
            } else {
              this.ConsoleNews('请完善地址!')
            }
          } else {
            this.ConsoleNews('请输入正确的手机号!')
          }
        } else {
          this.ConsoleNews('请完善联系方式!')
        }
      } else {
        this.ConsoleNews('请完善姓名!');
      }
    },

    ConsoleNews (text) {
      this.consoleNews = true;
      this.consoleNewsText = text;
    },

    // 判断是否是微信登录
    is_weixn () {
      var ua = navigator.userAgent.toLowerCase();
      if(ua.match(/MicroMessenger/i)=="micromessenger") {
          this.isWeixn = true;
      } else {
          this.isWeixn = false;
      }
    }
  },

  watch: {
    consoleNews (val) {
      if (val) {
        // statement
        setTimeout( () => {
          this.consoleNews = false;
        },2500)
      }
    }
  }
}
</script>
