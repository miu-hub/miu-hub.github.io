<template>
  <div id="root">
    <!-- 左侧导航 -->
    <div id="side_nav" ref="l_nav">
      <!-- logo -->
      <div id="side_logo">
        <div id="logo"></div>
        <p v-show="isfold">头条管理系统</p>
      </div>

      <!-- 分类组件 -->
      <!-- 遍历导航路由 -->
      <ul>
        <!-- 绑定点击触发的样式类 -->
        <!-- 使用绑定样式类来导入字体图标和文字/路由路径 -->
        <li
          :class="{ li_click: item.isclick }"
          v-for="(item, i) in rout"
          :key="i"
          @click="updata_style(i)"
        >
          <router-link :to="'/' + item.to"
            ><i :class="['iconfont', 'icon-' + item.class_icon]"></i>&nbsp;
            <p v-show="isfold">{{ item.text }}</p></router-link
          >
        </li>
      </ul>
    </div>

    <!-- 右侧导航栏 -->
    <div id="side_nav_r" ref="r_nav">
      <!-- 头部导航 -->
      <div id="header_nav">
        <!-- 公司信息 -->
        <div id="firm">
          <i
            :class="{
              iconfont: true,
              'icon-zhedie': isfold,
              'icon-zhedie2': !isfold,
            }"
            @click="fold"
          ></i>
          &nbsp;
          <p>ABC公司制作的头条管理系统</p>
        </div>

        <!-- 用户信息区 -->
        <div id="user_info">
          <div id="head_port">
            <!-- 从接口处获得头像信息 -->
            <img src="./demo_head.jpg" alt="" v-show="is_true" />
            <img :src="user.photo" alt="" v-show="is_false" />
          </div>
          &nbsp;&nbsp;
          <!-- 跳转方法 -->
          <!-- 从接口处获得用户明信息--------！！！未实现判断动态展示数据 -->
          <p @click="push" v-show="is_true">未登录</p>
          <p v-show="is_false">{{ user.name }}</p>
          &nbsp;&nbsp;
          <i class="iconfont icon-linecar105" @click="user_f = !user_f"></i>

          <!-- 用户功能区 -->
          <div id="user_facility" v-show="user_f">
            <ul>
              <li>个人设置</li>
              <li @click="remove_user('退出成功', 'green')">退出登录</li>
            </ul>
          </div>
        </div>
      </div>

      <!-- 主体 -->
      <div id="main">
        <router-view> </router-view>
      </div>
    </div>
    <!-- 警示框 -->
    <transition name="warring">
      <div id="warring" v-show="isclick_warring">
        <p ref="war_color">{{ war_text }}</p>
      </div>
    </transition>
  </div>
</template>

<script>
// 引入axios中的用户信息
import { user_info } from "@/apis/user";
export default {
  name: "layout",
  data() {
    return {
      // 导航栏的详细配置
      // to：路由路径
      // class_icon---所需的字体图标
      // text----展示的文字内容
      // isclick----决定点击后li的样式
      rout: [
        { to: "", class_icon: "shouye", text: "首页", isclick: false },
        {
          to: "conest",
          class_icon: "rizhimingxi",
          text: "内容管理",
          isclick: false,
        },
        {
          to: "matter",
          class_icon: "icon_huabanfuben",
          text: "素材管理",
          isclick: false,
        },
        {
          to: "issue",
          class_icon: "tubiao-",
          text: "发布文章",
          isclick: false,
        },
        {
          to: "comment",
          class_icon: "pinglun",
          text: "评论管理",
          isclick: false,
        },
        {
          to: "fans",
          class_icon: "weibiaoti-",
          text: "粉丝管理",
          isclick: false,
        },
        {
          to: "user",
          class_icon: "shezhi",
          text: "个人设置",
          isclick: false,
        },
      ],

      // 用户信息
      user: {},

      // 用户下拉功能
      user_f: false,

      // 登录状态
      is_true: true,
      is_false: false,

      // 折叠状态
      isfold: true,
      // 警告状态
      isclick_warring: false,
      // 警告框的文本状态
      war_text: "",
      war_color: "",
    };
  },

  // 在组件构建完整时接发送一次get请求
  created() {
    this.getUserInfo();
    // this.$router.push("/");
  },
  //   在虚拟Dom编译之后来使用总线获取数据
  mounted() {
    // 组件刷新时触发路径校验-----刷新页面导航栏仍有停留样式
    let path = this.$route.path.substring(1);
    this.rout.forEach((item, i) => {
      if (item.to == path) {
        // 让该路径下的组件显示被点击样式
        item.isclick = true;
      }
    });
    //   this.$bus总线
    // 为总线绑定一个事件su，并接受回调函数返回参数data
    this.$bus.$on("updata_user", () => {
      //  当个人设置触发提交数据的全局总线上的方法时会触发这里的更新请求
      this.getUserInfo();
    });
  },

  //   在vue实例销毁之前将$bus上的绑定事件销毁
  // 注意：千万不要什么都不写，，这要会销毁$bus上的所有事件
  beforeDestroy() {
    this.$bus.$off("updata_user");
  },

  methods: {
    // 鼠标点击路由导航
    updata_style(i) {
      //i------获取的点击到元素的序号
      const rout = this.rout;

      // 使用foreach方法来遍历路由信息-----利用排他案例来改变除点击元素的样式
      rout.forEach((item) => {
        item.isclick = false;
      });

      // 点击的元素的样式状态改变
      rout[i].isclick = true;
    },

    // 跳转方法----在未登录状态下跳转至登录页
    push() {
      this.$router.push({
        name: "login",
      });
    },

    //请求用户信息方法
    getUserInfo() {
      // 在首页用户信息处使用vuex中的用户令牌
      let tokens = this.$store.state.token;

      // 将数据通过本地存储来避免因为刷新首页而导致的用户信息缺失
      if (tokens) {
        localStorage.setItem("token", tokens);
      }
      // 获取用户令牌
      let token = localStorage.getItem("token");

      user_info(token)
        // 请求成功则值渲染
        .then((data) => {
          this.is_true = false;
          this.is_false = true;
          this.user = data.data.data;
        })
        //请求失败时
        .catch((err) => {
          this.is_true = true;
          this.is_false = false;
        });
    },

    // 退出登录功能
    remove_user(text, color) {
      localStorage.removeItem("token");
      // 调用弹出警告方法
      this.warring_click(text, color);
      // 3s后跳转
      setTimeout(() => {
        this.$router.push("/login");
      }, 3000);
    },

    //折叠导航栏
    fold() {
      if (this.isfold) {
        this.isfold = false;
        this.$refs.l_nav.style.width = "5%";
        this.$refs.r_nav.style.width = "95%";
      } else {
        this.isfold = true;
        this.$refs.l_nav.style.width = "16%";
        this.$refs.r_nav.style.width = "84%";
      }
    },
    // 警告框方法
    warring_click(text, color) {
      // 更改警告框文本
      this.war_text = text;
      // 更改警告文本颜色
      this.$refs.war_color.style.color = color;
      // 显示警告框
      this.isclick_warring = true;
      // 警告框隐藏
      setTimeout(() => {
        this.isclick_warring = false;
      }, 2000);
    },
  },
};
</script>

<style lang="less" scoped>
#root {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;

  // 警示框
  #warring {
    width: 300px;
    height: 50px;
    position: fixed;
    top: 50px;
    left: 45%;
    border-radius: 10px;
    background-color: #fff;
    box-shadow: 0px 0px 10px 0px #666;

    p {
      width: 100%;
      height: 50px;
      text-align: center;
      font-size: 20px;
      font-family: "kaiti";
      font-weight: 700;
      color: rgb(238, 116, 116);
      line-height: 50px;
    }
  }

  // 左侧
  #side_nav {
    float: left;
    width: 16%;
    height: 100%;
    background-color: rgb(29, 17, 63);

    // logo
    #side_logo {
      display: flex;
      width: 100%;
      height: 15%;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      border-bottom: 1px solid #fff;

      #logo {
        width: 80px;
        height: 80px;
        background-image: url("./head_logo.png");
        background-size: contain;
        background-repeat: no-repeat;
      }

      p {
        color: #fff;
        font-size: 25px;
        font-weight: 700;
        width: 180px;
        height: 50px;
        line-height: 50px;
      }
    }

    ul {
      margin-top: 10px;
      width: 100%;
      height: 90%;
      // 路由导航
      li {
        width: 100%;
        height: 10%;
        line-height: 88px;
        font-size: 18px;
        cursor: pointer;

        p {
          display: inline;
          // margin-left: 10px;
        }
        a {
          display: block;
          padding-left: 25%;
          width: 100%;
          height: 100%;
          color: #fff;
        }
        // 鼠标经过
        &:hover {
          a {
            color: yellow;
          }
        }
      }

      // 路由点击的样式
      .li_click {
        background-color: rgba(0, 0, 0, 0.3);
        a {
          color: red;
        }
      }
    }
  }

  // 右侧
  #side_nav_r {
    float: left;
    width: 84%;
    height: 100%;

    // 右侧头部
    #header_nav {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      height: 15%;
      background-color: #fff;
      border-bottom: 1px solid #666;

      // 公司信息
      #firm {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        width: 30%;
        height: 40%;

        i {
          font-size: 26px;
          font-weight: 700;
        }

        p {
          color: rgb(29, 17, 63);
          font-size: 26px;
          font-weight: 700;
        }
      }

      // 用户信息
      #user_info {
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 15%;
        height: 60%;

        // 用户头像
        #head_port {
          width: 50px;
          height: 50px;
          cursor: pointer;

          img {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            border: 1px solid #666;
          }
        }

        // 下拉用户信息
        i,
        p {
          font-size: 20px;
          cursor: pointer;
          color: #666;
        }

        // 用户下拉设置
        #user_facility {
          position: absolute;
          left: 40%;
          top: 100%;
          width: 40%;
          height: 100%;
          background-color: #fff;

          ul {
            width: 100%;
            height: 100%;
            border: 1px solid #ccc;
            li {
              width: 100%;
              height: 50%;
              border: 0;
              color: #666;
              text-align: center;
              line-height: 42px;
              cursor: pointer;
              &:last-child {
                border-top: 1px solid #000;
              }
            }

            li:hover {
              background-color: #ccc;
              color: blue;
            }
          }
        }
      }
    }

    // 展示区
    #main {
      width: 100%;
      height: 100%;
      background-color: #fff;
    }
  }

  .check_font {
    font-size: 20px;
  }
}

.warring-enter-active {
  animation: 1.5s show;
}
.warring-leave-active {
  animation: 0.5s show linear reverse;
}

@keyframes show {
  0% {
    top: 0px;
    opacity: 0;
  }
  100% {
    top: 50px;
    opacity: 1;
  }
}
</style>