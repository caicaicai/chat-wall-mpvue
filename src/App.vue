<script>
import { mapState, mapActions } from "vuex";
import tools from "@/utils/tools";
import weRequest from "@/utils/request";

export default {
  computed: {
    ...mapState([
      "appIMDelegate",
      "bcgImgList",
      "bcgImgIndex",
      "barBgColor",
      "bcgImg"
    ])
  },
  methods: {
    ...mapActions(["update"]),
    setNavigationBar(bcgImgIndex, barBgColor, bcgImg) {
      // 更新store
      this.update({ bcgImgIndex, barBgColor, bcgImg });
      wx.setNavigationBarColor({
        frontColor: "#ffffff",
        backgroundColor: barBgColor
      });
    }
  },
  onLaunch(options) {
    console.log('onLaunch')
    // 登录 获取session 存储到storage 用于解密数据
    weRequest.login(() => {
      // 登录之后的回调
    });

    // 设置bar颜色 isApp
    wx.getStorage({
      key: "bcgImgIndex",
      success: res => {
        let bcgImgIndex = res.data;
        let barBgColor = this.bcgImgList[bcgImgIndex].barBgColor;
        let bcgImg = this.bcgImgList[bcgImgIndex].src;
        this.setNavigationBar(bcgImgIndex, barBgColor, bcgImg);
      },
      fail: () => {
        let randomIndex = tools.getRandomNum(0, this.bcgImgList.length-1);

        let bcgImgIndex = randomIndex;
        let barBgColor = this.bcgImgList[bcgImgIndex].barBgColor;
        let bcgImg = this.bcgImgList[bcgImgIndex].src;
        this.setNavigationBar(bcgImgIndex, barBgColor, bcgImg);

        // 存储bcgImgIndex
        wx.setStorage({
          key: "bcgImgIndex",
          data: bcgImgIndex
        });
      }
    });

    // 获取系统信息
    wx.getSystemInfo({
      success: res => {
        let systemInfo;
        systemInfo = res;
        systemInfo.isIPhoneX = /iphonex/gi.test(res.model.replace(/\s+/, ""));
        console.log(systemInfo);

        // 存储系统信息
        this.update({ systemInfo });
        try {
          wx.setStorageSync("systemInfo", systemInfo);
        } catch (e) {}
      }
    });

    this.appIMDelegate.onLaunch(options);
  }
};
</script>
<style lang="scss">
Page {
  background: #f4f6f9;
}
::-webkit-scrollbar {
  display: none;
}
.iphonex-padding {
  padding-bottom: 68rpx;
}
</style>
