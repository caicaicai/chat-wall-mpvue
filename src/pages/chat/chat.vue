<template>
  <div class="container-chat"
       @click="hideMenus">
    <!-- <chat-status :chatStatue="chatStatue"
                 :chatStatusContent="chatSt
                 atusContent"></chat-status> -->
    <img class="bcg"
         :src="bcgImg"
         mode='aspectFill' />
    <scroll-view class="scroll-view"
                 :style="{'height':systemInfo.windowHeight+'px'}"
                 scroll-y="true"
                 :scroll-top="scrollTopVal"
                 scroll-with-animation="true"
                 @scroll="onScroll"
                 @scrolltoupper="onScrollToUpper"
                 @scrolltolower="onScrollToLower"
                 @click="resetInputStatus">
      <chat-item v-for="(item, index) in chatItems"
                 :item="item"
                 :length="chatItems.length"
                 :index="index"
                 :key="index"></chat-item>
      <!-- 底部占位 -->
      <div class="bottom-block" :style="{'height':inputObj.chatInputShowExtra ? '400rpx' :'200rpx'}"></div>
    </scroll-view>

    <!-- 菜单 -->
    <animation-menus ref="menus"></animation-menus>
    
    <chat-input :inputPlaceHolder="inputPlaceHolder" @sendMsgSuper="sendMsgSuper"></chat-input>
    <van-toast id="van-toast" />

  </div>
</template>
<script>
import vue from "vue";
import { mapState, mapActions } from "vuex";
import Toast from "@/../static/vant/toast/toast";
import config from "@/config/config";
import tools from "@/utils/tools";
import weRequest from "@/utils/request";
import IMOperator from "./im-operator";
import UI from "./ui";
import MsgManager from "./msg-manager";
import { inputPlaceHolder } from "@/config/constant";
import * as chatInputTools from "@/components/chat-input/chat-input-tools";

import ChatStatus from "@/components/chat-page/chat-status";
import ChatItem from "@/components/chat-page/chat-item";
import ChatInput from "@/components/chat-input/chat-input";
import AnimationMenus from "@/components/menus/menus";
export default {
  mpType: "page",
  data() {
    return {
      isDragScrollView: false,// 是否拖动scrollview
      chatItems: [],
      allChatLength: 0,
      latestPlayVoicePath: "",
      showVoicePart: config.showVoicePart,
      isAndroid: true,
      chatStatue: "open",
      scrollTopVal: 0,
      inputPlaceHolder:
        inputPlaceHolder[tools.getRandomNum(1, inputPlaceHolder.length - 1)]
    };
  },
  components: {
    ChatItem,
    ChatStatus,
    ChatInput,
    AnimationMenus
  },
  computed: {
    ...mapState([
      "inputObj",
      "appInfo",
      "systemInfo",
      "barBgColor",
      "bcgImg",
      "appIMDelegate"
    ])
  },
  // 页面隐藏关闭socket
  onHide() {
    this.appIMDelegate.iIMHandler.closeConnection();
  },
  // 页面显示连接socket
  onShow() {
    // 获取聊天历史记录 最近10条
    
    this.appIMDelegate.onShow({ appInfo: this.appInfo });

    
  },
  onLoad(options) {
    // 设置bar颜色
    wx.setNavigationBarColor({
      frontColor: "#ffffff",
      backgroundColor: this.barBgColor
    });
    // this.initData();
    this.imOperator = new IMOperator(this, {
      appInfo: this.appInfo,
      timeStr: "",
      timestamp: null,
      type: "text",
      appIMDelegate: this.appIMDelegate
    });
    this.UI = new UI(this);
    this.msgManager = new MsgManager(this);
    let self = this;
    this.imOperator.onSimulateReceiveMsg(msg => {
      console.log("2333333333333")
      console.log(msg)
      this.msgManager.showMsg({ msg });
    });
  },
  watch: {
    // 监听聊天记录条数 只保留20条
    chatItems(val, oldVal) {
      console.error('####################')
      console.log('val.length')
      console.log(val.length)
      console.log(oldVal.length)
      if (!(oldVal.length < 30)) {
        val.splice(0, 10);
      }
    }
  },
  methods: {
    ...mapActions(["updateInputObj"]),
    // 自动滚动到底部
    autoScrollToLower() {
      this.scrollTopVal = this.allChatLength * 999
      
    },
    onScroll(event) {
      let deltaY = event.mp.detail.deltaY;

      console.log(deltaY)
    },
    onScrollToUpper() {
      console.log('onScrollToUpper')
    },
    onScrollToLower() {
      console.log('onScrollToLower')
    },
    hideMenus() {
      this.$refs.menus.menuHide();
      this.updateInputObj({"chatInputShowExtra": false})
    },
    initData() {
      let self = this;
      // chatInputTools.init(this, {
      //   systemInfo: this.systemInfo,
      //   minVoiceTime: 1,
      //   maxVoiceTime: 60,
      //   startTimeDown: 56,
      //   format: "mp3", //aac/mp3
      //   sendButtonBgColor: this.barBgColor,
      //   sendButtonTextColor: "white",
      //   extraArr: [
      //     {
      //       picName: "choose_picture",
      //       description: "照片"
      //     },
      //     {
      //       picName: "take_photos",
      //       description: "拍摄"
      //     },
      //     {
      //       picName: "close_chat",
      //       description: "自定义功能"
      //     }
      //   ]
      //   // tabbarHeigth: 48
      // });

      this.pageHeight = this.systemInfo.windowHeight;
      this.isAndroid = this.systemInfo.system.indexOf("Android") !== -1;

      self.textButton();
      self.extraButton();
      // self.voiceButton();
    },
    sendMsgSuper(msgObj) {
      console.log('chat send content')
      this.msgManager.sendMsg(msgObj);
    },
    textButton() {
      // chatInputTools.setTextMessageListener(e => {
      //   let content = e.mp.detail.value;
      //   this.msgManager.sendMsg({ type: IMOperator.TextType, content });
      // });
      console.log('this.inputObj')
      console.log(this)
      console.log(this.inputObj)
      
    },
    voiceButton() {
      chatInputTools.recordVoiceListener((res, duration) => {
        let tempFilePath = res.tempFilePath;
        this.msgManager.sendMsg({
          type: IMOperator.VoiceType,
          content: tempFilePath,
          duration
        });
      });
      chatInputTools.setVoiceRecordStatusListener(status => {
        this.msgManager.stopAllVoice();
      });
    },
    //模拟上传文件，注意这里的cbOk回调函数传入的参数应该是上传文件成功时返回的文件url，这里因为模拟，我直接用的savedFilePath
    simulateUploadFile({ savedFilePath, duration, itemIndex, success, fail }) {
      this.upLoadPic(savedFilePath, res => {
        res.data = res.data.replace(/http/g, "https");
        let data = JSON.parse(res.data);
        if (data.msg === "ok" && data.data.images) {
          // 返回所有图片
          success && success(data.data.images);
        }
      });
    },
    extraButton() {
      let self = this;
      chatInputTools.clickExtraListener(e => {
        let chooseIndex = parseInt(e.currentTarget.dataset.index);
        if (chooseIndex === 2) {
          self.myFun();
          return;
        }
        wx.chooseImage({
          count: 1, // 默认9
          sizeType: ["compressed"],
          sourceType: chooseIndex === 0 ? ["album"] : ["camera"],
          success: res => {
            this.msgManager.sendMsg({
              type: IMOperator.ImageType,
              content: res.tempFilePaths[0]
            });
          }
        });
      });
      chatInputTools.setExtraButtonClickListener(dismiss => {
        console.log("Extra弹窗是否消失", dismiss);
        this.showExtraPart = dismiss;
      });
    },
    upLoadPic(path, success, fail) {
      wx.showLoading({
        title: "努力上传中..."
      });
      wx.uploadFile({
        url: config.uploadPicUrl, //新浪微博图床
        filePath: path,
        name: "file",
        // formData: {
        //   user: "test"
        // },
        success(res) {
          wx.hideLoading();
          success && success(res);
        },
        fail(error) {
          fail && fail(error);
        }
      });
    },
    /**
     * 自定义事件
     */
    myFun() {
      wx.showModal({
        title: "小贴士",
        content: "演示更新会话状态",
        confirmText: "确认",
        showCancel: true,
        success: res => {
          if (res.confirm) {
            this.msgManager.sendMsg({ type: IMOperator.CustomType });
          }
        }
      });
    },
    resetInputStatus() {
      chatInputTools.closeExtraView();
    },
    sendMsg({ content, itemIndex, success }) {
      console.log("--------------")
      console.log(content)
      // 发送消息后修改placeholder
      content &&
        content.type === "text" &&
        (this.inputPlaceHolder =
          inputPlaceHolder[tools.getRandomNum(1, inputPlaceHolder.length - 1)]),
        this.imOperator.onSimulateSendMsg({
          content,
          success: msg => {
            console.log("------gengxin--------")
            console.log(msg)
            this.UI.updateViewWhenSendSuccess(msg, itemIndex);
            success && success(msg);
          },
          fail: () => {
            this.UI.updateViewWhenSendFailed(itemIndex);
          }
        });
    },
    /**
     * 重发消息
     * @param e
     */
    resendMsgEvent(e) {
      const itemIndex = parseInt(e.currentTarget.dataset.resendIndex);
      const item = this.data.chatItems[itemIndex];
      this.UI.updateDataWhenStartSending(item, false, false);
      this.msgManager.resend({ ...item, itemIndex });
    }
  }
};
</script>
<style lang="scss" scoped>
.container-chat {
  background-color: #efefef;
  .bcg {
    position: fixed;
    // z-index: 2;
    height: 100%;
    width: 100%;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    /* background: #40a7e7; */
    /* background: linear-gradient(to bottom, #73C6F1, #50B5EC); */
  }
  .scroll-view {
    width: 100%;
    display: flex;
    flex-direction: column;
    // margin-top: 54rpx;
    // 解决安卓scroll-view滑动卡顿
    -webkit-overflow-scrolling: touch;
    box-sizing: border-box;
    .bottom-block {
      width: 100%;
    }
  }
}
</style>
