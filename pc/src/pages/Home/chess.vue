<template>
  <div class="play">
    <div class="playList">
      <div style="height: 36px">
        <!-- 公告 -->
        <div :class="noticeClass" @click="openNotice">
          <Notice></Notice>
        </div>
        <!-- 公告弹窗 -->
        <dialog-notice v-if="openDialog" @close="closeNotice"></dialog-notice>
      </div>
      <!-- 顶部背景图片 -->
      <div class="bg">
        <img loading="lazy" class="img" v-lazy="$config.getLocaleImg('listbg2','jpg')" alt="">
      </div>
      <!-- 游戏二级列表 -->
      <ul class="card-vendor">
        <!-- <li class="vendor-item current" v-for="(item,index) in curMenuList" :key='index' @click="clickTab(index)" :class="index == ind ? 'setColor':''">{{item.name}}</li> -->
        <li
          class="vendor-item current"
          v-for="(item, index) in curMenuList"
          :key="index"
          @click="clickTab(item)"
          :class="item.ids == dataInfo.id ? 'setColor' : ''"
        >
          {{ item.name }}
        </li>
      </ul>

      <!-- 搜索轮播 -->
      <div class="searchSwiper" v-if="window.projectImgUrl !='funw'">
        <!-- 轮播 -->
        <div class="swiper"></div>
        <!-- 搜索 -->
        <div class="search">
          <!-- <swiper :options="swiperOption" style="height: 357px">
            <swiper-slide
              class="item"
              v-for="(item, index) in list"
              :key="index"
            >
              <img
                loading="lazy"
                style="width: 100%"
                class="img"
                v-lazy="item.img"
              />
            </swiper-slide>
          </swiper> -->
          <!--以下看需要添加-->
          <!-- <div class="swiper-button-next"></div>
                    <div class="swiper-button-prev"></div> 
                    <div class="swiper-pagination"></div> -->
          <!-- <img loading="lazy" style="width:100%;" src="@/assets/image/pubilc/chess-searchbg.jpg" alt=""> -->
          <!-- <div style="width:100%;height:36px;">
                        <div style="width:40px;height:100%;font:16px/36px normal;float:right;">搜索</div>
                        <el-input size="normal" style="width:150px;height:36px;float:right;"></el-input>
                    </div> -->
        </div>
      </div>
      <!-- 游戏展示列表 -->
      <div class="showPlayList">
        <!--鼠标经过显示进入游戏动画HTML   开始-->
        <ul style="overflow: hidden; margin-top: 15px">
          <li
            class="ele-card-box has-link"
            v-for="(item1, ind) in gameList"
            :key="ind"
            @click="getToken(item1, 1)"
          >
            <div class="img-wrap" :class="{ 'img-wrap1': item1.status == 0 }">
              <img v-if="ind===0" :src="$config.imgHost + item1.pictureUrl" />
              <img v-else loading="lazy" v-lazy="$config.imgHost + item1.pictureUrl" />
              <div class="mask">
                <div class="mask-word">
                  {{ item1.status == 1 ? $t("进入游戏") : $t("维护中") }}
                </div>
              </div>
            </div>
            <div class="ele-card-bottom">{{ item1.name }}</div>
          </li>
        </ul>
        <div
          style="
            display: flex;
            width: 1200px;
            height: 400px;
            color: white;
            justify-content: center;
            align-items: center;
            letter-spacing: 2px;
            font: 18px/36px normal;
          "
          v-if="dataInfo.total == 0"
        >
          {{$t('暂无数据')}}
        </div>
        <!--鼠标经过显示进入游戏动画HTML   结束-->
      </div>
      <div>
        <el-pagination
          background
          :page-size="dataInfo.pageSize"
          :current-page="dataInfo.curPage"
          @current-change="changePage"
          layout="prev, pager, next"
          :total="dataInfo.total"
        >
        </el-pagination>
      </div>
    </div>
  </div>
</template>
<script>
import api from "../../utils/api"; //接口名字
// 公告
import Notice from "../../components/index/notice";
// 公告弹窗
import dialogNotice from "../../components/index/dialogNotice";
export default {
  components: {
    Notice,
    dialogNotice,
  },
  data() {
    return {
      window,
      menuList: [], // 总列表
      curMenuList: null, // 当前选中二级列表
      gameList: [], // 游戏列表
      // 重要参数
      dataInfo: {
        pid: "", // 父id
        id: "", //子id
        type: "", // 类型
        curPage: 1, //当前页
        pageSize: 18, // 展示数量
        total: 0, //游戏列表总数量
      },

      openDialog: false,
      noticeSwitch: false,
      data: [],
      ind: null, //点击li的索引,
      img: require("@/assets/image/gameImg/Game_49220.png"),
      gameList: [
        {
          img: require("@/assets/image/gameImg/Game_49220.png"),
          title: this.$t("五人牛牛"),
        },
      ],
      swiperOption: {
        //swiper3
        autoplay: 3000,
        speed: 1000,
        // slidesPerView: 3,
        // spaceBetween: 30,
        loop: true,
        // prevButton:'.swiper-button-prev',
        // nextButton:'.swiper-button-next',
        // pagination: '.swiper-pagination',
      },
      list: [
        {
          img: require("@/assets/image/pubilc/chess-searchbg.jpg"),
        },
        {
          img: require("@/assets/image/pubilc/chess-searchbg1.jpg"),
        },
        {
          img: require("@/assets/image/pubilc/chess-searchbg2.jpg"),
        },
        {
          img: require("@/assets/image/pubilc/chess-searchbg3.jpg"),
        },
        {
          img: require("@/assets/image/pubilc/chess-searchbg4.jpg"),
        },
        {
          img: require("@/assets/image/pubilc/chess-searchbg5.jpg"),
        },
      ],
    };
  },
  mounted() {
    let _this = this;
    // 滚动公告
    window.onscroll = function () {
      if (document.documentElement.scrollTop > 251) {
        _this.noticeSwitch = true;
      } else {
        _this.noticeSwitch = false;
      }
    };
    // 赋值 总列表
    this.menuList = JSON.parse(localStorage.getItem("ALLMENUE_EXCEPT_FISH"));

    // 获取传参数据
    let { pid, id, type } = this.$route.query;
    this.dataInfo.pid = pid;
    this.dataInfo.id = id;
    this.dataInfo.type = type;
    this.curMenu = this.menuList.filter((v) => {
      return v.id == this.$route.query.pid;
    });
    this.curMenuList = this.curMenu[0].children;
    this.getGameList();
  },
  computed: {
    noticeClass: function () {
      return {
        notice: !this.noticeSwitch,
        fixed: this.noticeSwitch,
      };
    },
  },
  watch: {
    "$route.query"(val) {
      let { pid, id, type } = val;
      this.dataInfo.pid = pid;
      this.dataInfo.id = id;
      this.dataInfo.type = type;
      this.dataInfo.curPage = 1;
      this.curMenu = this.menuList.filter((v) => {
        return v.id == this.$route.query.pid;
      });
      this.curMenuList = this.curMenu[0].children;
      this.getGameList();
    },
  },
  methods: {
    clickTab(item) {
      let { parentId: pid, ids: id, type } = item;
      this.dataInfo.pid = pid;
      this.dataInfo.id = id;
      this.dataInfo.type = type;
      this.dataInfo.curPage = 1;
      this.getGameList();
    },
    openNotice() {
      this.openDialog = true;
    },
    closeNotice() {
      this.openDialog = false;
    },
    // 修改页码
    changePage(val) {
      this.dataInfo.curPage = val;
      this.getGameList();
    },
    // 获取游戏列表数据
    getGameList() {
      let self = this;
      let { pid, id, type, curPage, pageSize } = this.dataInfo;
      if (type == 1) {
        self.$http.pnPost(
          self.$api.vendorGame,
          {
            currentPage: curPage,
            pageSize: pageSize,
            gameKindId: pid,
            vendorId: id,
          },
          true,
          (res) => {
            self.gameList = res.data.data.list;
            self.dataInfo.total = res.data.data.total;
          }
        );
      } else if (type == 3) {
        self.$http.pnPost(
          self.$api.getGameByIds,
          {
            currentPage: curPage,
            pageSize: pageSize,
            gameKindId: pid,
            ids: id,
          },
          true,
          (res) => {
            self.gameList = res.data.data.list;
            self.dataInfo.total = res.data.data.total;
          }
        );
      }
    },
    // 进入游戏
    getToken: async function (req, index) {
      // return
      let self = this;
      var issafariBrowser =
        /Safari/.test(navigator.userAgent) &&
        !/Chrome/.test(navigator.userAgent);

      if (!self.$common.getUser()) {
        // self.showLogin = true;
        // self.$message.warning(this.$t("请登录"));
        this.$common.openLogin()
        return;
      }
      self.tenantId = self.$common.getUser().tenant_id;
      self.username = self.$common.getUser().username;
      self.uid = self.$common.getUser().user_id;
      let datas = {
        tenantId: self.tenantId,
        username: self.username,
        gameId: index == 2 ? req.ids : req.id,
        clientIp: self.$config.clientIp,
        memberId: self.uid,
        terminalType: 1,
      };
      self.$common.setGameRequestData(datas);

      const res = await self.$http.post(api.getToken, datas, true);
      if (res.code == 0) {
        self.gameUrl = res.data;
        if (req.openMode === 1) {
          // if(issafariBrowser) {
          //  self.winOpen.location.href = self.gameUrl;
          // }else{
          window.open(self.gameUrl);
          // }
        } else {
          window.open(self.gameUrl);
        }
      } else {
        if (req.openMode === 1) {
          window.open("/error.html?type=1");
          // self.winOpen.location.href = '/error.html?type=1';
        }

        if (req.status === 0) {
          self.$message.error(this.$t("维护中"));
        } else {
          self.$message.error(this.$t("进入游戏失败，请稍后重试"));
        }
      }
    },
  },
};
</script>
<style>
.play .el-pagination.is-background {
  text-align: center;
  padding-top: 15px;
}
.play .el-pager li,
.play .el-pagination.is-background .btn-prev,
.play .el-pagination.is-background .btn-next {
  background-color: #2a2a2a !important;
}
.play .el-pager .number:hover,
.play .btn-prev:hover,
.play .btn-next:hover {
  color: #fff !important;
}
</style>
<style scoped lang="scss">
.fixed {
  position: fixed;
  left: 0;
  top: 130px;
  width: 100%;
  background-color: rgba(0, 0, 0, 0.85);
  z-index: 99;
  cursor: pointer;
}
.play {
  background-color: $activity-bg;
}
.playList {
  width: 1200px;
  margin: 0 auto;
}
.playList .bg {
  position: relative;
  min-width: 1920px;
  width: 100%;
  height: 260px;
  left: -360px;
  background-color: #fff;
}
.playList .bg .img {
  position: absolute;
  width: 100%;
        min-width: 1920px;
  left: 50%;
  top: 0;
  transform: translateX(-50%);
}
.searchSwiper {
  width: 1200px;
  /* height: 400px; */
  background-color: white;
}
.card-vendor {
  background: $game-tabBg;
  overflow: hidden;
}
.card-vendor .vendor-item {
  position: relative;
  float: left;
  width: 16.66%;
  height: 43px;
  line-height: 43px;
  padding: 0 10px;
  box-sizing: border-box;
  color: $game-textColor;
  font-size: 16px;
  text-align: center;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  cursor: pointer;
}
.card-vendor .vendor-item::before {
  position: absolute;
  top: 0;
  right: 0;
  width: 2px;
  height: 100%;
  content: "";
  background-color: $game-Bg;
  border-left: 1px solid $game-Rborder;
}
.card-vendor .vendor-item::after {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 1px;
  content: "";
  background-color: $game-Rborder;
  border-bottom: 2px solid $game-Bg;
}
.card-vendor .vendor-item:hover{
        color: $game-tabColor;
    }
   .playList .card-vendor .setColor{
        color: $game-tabColor;
    }
/*鼠标经过显示进入游戏动画CSS   开始*/
.has-link {
  cursor: pointer;
  float: left;
  width: 15.83%;
  margin: 0 1% 10px 0;
  background: rgba(44, 44, 44, 0.1);
  z-index: 10;
}
.has-link:nth-child(6n + 6) {
  margin-right: 0;
}
.has-link:hover .img-wrap .mask {
  opacity: 0.9;
}

.has-link .img-wrap1 .mask {
  opacity: 0.9 !important;
}
.ele-card-box .img-wrap {
  position: relative;
  text-align: center;
  padding-top: 100%;
}
.ele-card-box .img-wrap img {
  position: absolute;
  left: 0;
  top: 0;
  max-width: 100%;
  padding: 3%;
  box-sizing: border-box;
}
.ele-card-box .img-wrap .mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #29e5c1;
  background: linear-gradient(135deg, #29e5c1, #28bbcb 50%, #2772dc);
  opacity: 0;
  -moz-transition: opacity 0.5s ease-in-out;
  -o-transition: opacity 0.5s ease-in-out;
  transition: opacity 0.5s ease-in-out;
}
.ele-card-box .img-wrap .mask .mask-word {
  position: absolute;
  top: 50%;
  left: 0;
  width: 100%;
  height: 20px;
  line-height: 20px;
  padding-top: 70px;
  margin-top: -45px;
  text-align: center;
  letter-spacing: 1px;
  font-size: 18px;
  color: #fff;
  background: url("../../assets/image/gameImg/bordergame_play_icon.png")
    top no-repeat;
  background-size: auto 68%;
}
.ele-card-box .ele-card-bottom {
  background: #ddd;
  background: rgba(58, 61, 63, 0.3);
  padding: 0 4px;
  height: 32px;
  line-height: 32px;
  text-align: center;
  font-size: 17px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
  color: #fff;
  letter-spacing: 1px;
  text-decoration: none;
  overflow: hidden;
  text-overflow: ellipsis;
  -ms-text-overflow: ellipsis;
  white-space: nowrap;
}
/*鼠标经过显示进入游戏动画CSS   结束*/
</style>