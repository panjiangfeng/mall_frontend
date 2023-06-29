<template>
  <el-container>
    <el-header>
      <div class="userhead">
        <div>
          <p>账户</p>
        </div>
        <div class="sessionquit" @click="logout" style="cursor: pointer">
          quit
        </div>
      </div>
      <div class="line"></div>
      <div class="user">
        <div class="userimg">
          <img :src="host+this.user.userinfo.tximg"/>
        </div>
        <div class="hellouser">
          <p>{{ this.user.username }} , 你好：</p>
        </div>
      </div>
    </el-header>
    <el-main>
      <!--      选择按钮-->
      <div class="allchoose">
        <div
            class="userchoose"
            @click="choosechange('ordertag')"
            style="cursor: pointer"
        >
          <p>近期订单</p>
        </div>
        <div
            class="userchoose"
            @click="choosechange('carttag')"
            style="cursor: pointer"
        >
          <p>我的购物车</p>
        </div>
        <div
            class="userchoose"
            @click="choosechange('adresstag')"
            style="cursor: pointer"
        >
          <p>我的地址</p>
        </div>
        <div
            class="userchoose"
            @click="choosechange('OtherChoose')"
            style="cursor: pointer"
        >
          <p>OtherChoose</p>
        </div>
      </div>
      <!--      近期订单-->
      <div class="order" v-show="ordertag">
        <div
            class="orderitem"
            v-for="(item, index) in this.oderlist"
            :key="index"
        >
          <div class="goodsimg">
            <img :src="host+item.goodsimg"/>
          </div>
          <div class="content">
            <p>订单编号：{{ item.orderid }}</p>
            <p>商品名称：{{ item.goodstitle }}</p>
            <p>大小参数：{{ item.size }}</p>
            <p>送货地址：{{ item.adress }}</p>
            <p>联系人: {{ item.username }}</p>
            <p>联系方式：{{ item.email }}</p>
            <p>付款价格：{{ item.goodsprice }}</p>
            <!--            发货标记-->
            <img
                class="send"
                src="~assets/img/alreadsend.png"
                v-if="item.issend"
            />
          </div>
        </div>
        <div
            v-show="noorder"
            style="text-align: center; font-size: 1rem; margin-top: 20%"
        >
          您还没有订单呢，快去选购吧~
        </div>
      </div>
      <!--      我的购物车-->
      <div class="cart" v-show="carttag">
        <div
            class="cartitem"
            v-for="(item, index) in this.cartlist"
            :key="index"
        >
          <div class="goodsimg">
            <img :src="host+item.info.imglist[0].goodsimg"/>
          </div>
          <div class="content">
            <p>商品编号：{{ item.info.goodsid }}</p>
            <p>商品名称：{{ item.info.goodsname }}</p>
            <p>商品类别：{{ item.info.category }}</p>
            <p>商品标题：{{ item.info.goodstitle }}</p>
            <p>大小参数：{{ item.choosesize }}</p>
            <p>商品价格：{{ item.info.price }}</p>
            <p>购买数量：{{ 1 }}</p>
            <div class="remove-btn" @click="popCart(index)">移除</div>
          </div>
        </div>
        <div class="foot_item" v-show="this.cartlist.length!=0">
          <div class="price">总价格：{{ getTotalPrice }}</div>
          <div class="buy">
            <div class="buybtn" style="cursor: pointer;" @click="buy">
              购买
            </div>
          </div>
        </div>

        <div
            v-show="this.cartlist.length==0"
            style="text-align: center; font-size: 1rem; margin-top: 20%"
        >
          您还没有添加任何商品到购物车呢，快去选购吧~
        </div>
      </div>
      <!--     查看修改地址 -->
      <div class="adress" v-show="adresstag">
        <div class="saidhead">
          <p>送货：</p>
        </div>
        <div class="adressmain">
          <p>
            送货地址：<input
              type="text"
              v-model="adress"
              :placeholder="this.user.userinfo.adress"
          />
          </p>
          <div
              class="sessionquit"
              @click="changeadress"
              style="
              height: 1.7em;
              width: 55px;
              font-size: 0.8em;
              line-height: 1.7em;
            "
              v-show="this.adress != ''"
          >
            保存修改
          </div>
          <p>联系人:{{ this.user.username }}</p>
          <p>联系方式：{{ this.user.email }}</p>
        </div>
      </div>
      <!--     OtherChoose -->
      <div class="other" v-show="OtherChoose">
        <div style="width: 70%">
          <div style="margin: 20px 0 30px 10%">
            <p>Perfect shop , right?</p>
          </div>
          <div class="rate" style="cursor: pointer">
            <el-rate
                v-model="value"
                :texts="[
                'good',
                'nice',
                'perfect',
                'very perfect',
                'very very perfect',
              ]"
                :colors="[
                'rgba(255,129,152,0.1)',
                'rgba(255,129,152,0.4)',
                'rgba(255,129,152,0.8)',
              ]"
                show-text
                :change="ratechange"
            >
            </el-rate>
          </div>
        </div>
      </div>
    </el-main>
  </el-container>
</template>

<script>
import "assets/css/user_detaillELUI.css";
import {chexkrules} from "@/assets/mixins/checkrules";
import {editUserinfo} from "@/network/admin";
import {deepCopyAdvanced} from "@/common/utils";
import {multipleBuy, secondbuy} from "@/network/goods";

export default {
  name: "Userinfo",
  data() {
    return {
      noorder: false,
      host: "http://127.0.0.1:9000",
      user: {},
      oderlist: [],
      ordertag: true,
      adresstag: false,
      carttag: false,
      OtherChoose: false,
      nowchoose: "ordertag",
      adress: "",
      cartlist: [],
      value: null,
    };
  },
  computed: {
    getTotalPrice() {
      const data = this.cartlist
      let price = 0;
      for (let i = 0; i < data.length; i++) {
        price += data[i].info.price
      }
      return price;
    },
  },
  created() {
    //拿到user信息
    this.user = this.$store.state.user;
    this.cartlist = this.$store.state.cartlist;
    console.log(this.$store.state)
    //拿到需要处理的oderlist格式
    this.oderlist = deepCopyAdvanced(this.$store.state.user.orderlist);
    this.oderlist.reverse(); //顺序翻转
    if (this.oderlist.length == 0) this.noorder = true;
  },
  mixins: [chexkrules],
  methods: {
    logout() {
      window.sessionStorage.clear(); //清除用户的账户密码
      this.$store.state.user = {}; //清空vuex缓存
      this.$router.push("/login");
      this.$emit("loginsuccess");
    },
    choosechange(x) {
      //关闭上一个选项
      if (!!this.nowchoose) {
        this[this.nowchoose] = !this[this.nowchoose];
      }
      //开启点击选项
      this[x] = !this[x];
      //定义当前选项
      this.nowchoose = x;
    },
    changeadress() {
      if (
          this.adress != this.user.userinfo.adress &&
          this.checkadress(this.adress)
      ) {
        editUserinfo(this.user.id, this.user.userinfo.age, this.adress).then(
            (res) => {
              switch (res.data) {
                case "error":
                  this.$message.error("~修改操作失败~");
                  break;
                case "updatesuccess":
                  this.$message.success("~edit Success~");
                  this.$store.state.user = {};
                  break;
              }
            }
        );
      } else {
        this.$message.warning("请输入详细地址且与原址不同");
      }
    },
    ratechange() {
      console.log("1111111");
      console.log();
    },
    buy() {
      if (!this.$store.state.user.id) {
        this.$confirm("您没有登录哦，请先登录，再选购呦");
        this.$router.push("/home");
        // this.$router.push("/home/login")     无法跳转无关子路由
      } else {
        if (this.choosesize != "") {
          // 默认付款成功  发送请求实现用户的订单添加到对应的用户订单表中
          // console.log(this.$store.state.user.id+'===='+this.id+"======"+this.choosesize)
          const data = this.cartlist;
          let goods = [];
          for (let i = 0; i < data.length; i++) {
            goods.push({
              id: data[i].id,
              choosesize: data[i].choosesize
            })
          }

          multipleBuy(this.$store.state.user.id, goods).then(
              (res) => {
                if (res.data == "success") {
                  this.$message.success("您的订单下单成功~请查收~");

                  this.cartlist = [];
                  this.$store.state.cartlist = [];
                  setTimeout(()=>{
                    location.reload()
                  },500)
                }
              }
          );
        } else {
          this.$message.warning("~购买失败~");
        }
      }
    },
    popCart(index){
      this.cartlist.splice(index, 1);
      this.$store.state.cartlist.splice(index, 1);
    }
  },
};
</script>

<style lang="less" scoped>
.sessionquit {
  width: 100px;
  height: 50px;
  border-radius: 50px;
  border: 0.5px solid rgba(12, 12, 12, 0.1);
  /*border: .1px solid #8c8c8c;*/
  box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
  display: flex;
  align-self: center;
  justify-content: center;
  line-height: 50px;
  font-size: 1rem;
  font-weight: 300;
  //font-family: monospace;
  //transition: .6s;   控制的是内部元素的变形时间
}

.sessionquit:hover {
  border: 0.1px solid #8c8c8c;
}

.el-header {
  height: auto !important;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  //font-family: monospace;
  background-color: rgba(241, 246, 209, 0.3);

  .userhead {
    margin-top: 3rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 2rem;
    width: 100%;
    padding: 0 20px;
  }

  .user {
    width: 100%;
    display: flex;

    .userimg {
      width: 120px;
      height: 120px;
      border-radius: 50%;
      overflow: hidden;

      img {
        width: 100%;
      }
    }

    .hellouser {
      width: 100%;
      height: 70px;
      display: flex;
      align-items: center;
      justify-content: left;
      padding-left: 20px;
      font-size: 2rem;
      font-weight: 500;
    }
  }
}

.el-main {
  //font-family: monospace;
  .allchoose {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    padding: 0 3rem;

    .userchoose {
      width: 10rem;
      height: 3rem;
      border: 0.5px solid rgba(12, 12, 12, 0.1);
      box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
      border-radius: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.2rem;
      font-weight: 200;
    }

    .userchoose:hover {
      border: 0.1px solid #8c8c8c;
    }
  }

  .order {
    padding: 30px;

    .orderitem {
      padding: 10px;
      border: 1px solid rgba(255, 129, 152, 0.3);
      border-radius: 15px;
      width: 100%;
      margin: 10px 0;
      box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
      display: flex;
      align-items: center;

      .goodsimg {
        margin: 15px;
        width: 170px;
        //height: 170px;
        //overflow: hidden;
        //border-radius: 10%;
        img {
          width: 100%;
          border-radius: 10%;
        }
      }

      .content {
        position: relative;
        width: 100%;
        font-size: 1rem;
        line-height: 1.5rem;
        //英文和数字这种连续文本也会被迫换行
        word-wrap: break-word;
        //文本默认会自动换行
        white-space: normal;

        .send {
          width: 70px;
          position: absolute;
          top: 50%;
          right: 10%;
        }
      }
    }
  }

  .cart {
    padding: 30px;

    .cartitem {
      padding: 10px;
      border: 1px solid rgba(255, 129, 152, 0.3);
      border-radius: 15px;
      width: 100%;
      margin: 10px 0;
      box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
      display: flex;
      align-items: center;

      .goodsimg {
        margin: 15px;
        width: 170px;
        //height: 170px;
        //overflow: hidden;
        //border-radius: 10%;
        img {
          width: 100%;
          border-radius: 10%;
        }
      }

      .content {
        position: relative;
        width: 100%;
        font-size: 1rem;
        line-height: 1.5rem;
        //英文和数字这种连续文本也会被迫换行
        word-wrap: break-word;
        //文本默认会自动换行
        white-space: normal;

        .remove-btn {

          width: 5rem;
          height: 2rem;
          border: 0.5px solid rgba(12, 12, 12, 0.1);
          box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
          border-radius: 10px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 1.2rem;
          font-weight: 200;
        }

        .remove-btn:hover {
          border: 0.1px solid #8c8c8c;
        }

      .send {
        width: 70px;
        position: absolute;
        top: 50%;
        right: 10%;
      }
    }
  }

  .foot_item {
    padding: 10px;
    border: 1px solid rgba(255, 129, 152, 0.3);
    border-radius: 15px;
    width: 100%;
    margin: 10px 0;
    box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
    display: flex;
    align-items: center;
    justify-content: space-between;

    .price {
      padding: 2rem;
      width: 55%;
      font-size: 32px;

    }

    .buy {
      width: 43%;

      .buybtn {
        width: 10rem;
        height: 3rem;
        border: 0.5px solid rgba(12, 12, 12, 0.1);
        box-shadow: 7px 0px 2px rgba(12, 12, 12, 0.1);
        border-radius: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 1.2rem;
        font-weight: 200;
      }

      .buybtn:hover {
        border: 0.1px solid #8c8c8c;
      }
    }

    .goodsimg {
      margin: 15px;
      width: 170px;
      //height: 170px;
      //overflow: hidden;
      //border-radius: 10%;
      img {
        width: 100%;
        border-radius: 10%;
      }
    }
  }
}

.adress {
  display: flex;
  width: 100%;
  align-items: center;
  height: 100%;
  margin-top: 6rem;

  .saidhead {
    width: 30%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2.5rem;
    font-weight: 500;
  }

  .adressmain {
    width: 70%;
    padding-top: 3rem;
    line-height: 3rem;
    font-size: 2rem;
    //英文和数字这种连续文本也会被迫换行
    word-wrap: break-word;

    input {
      background-color: rgba(255, 255, 255, 0);
      border-radius: 30px;
      border: 0px;
      outline: none;
    }
  }
}

.other {
  width: 100%;
  margin-top: 7rem;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.7rem;

  /deep/ .el-rate__icon {
    font-size: 4rem;
  }

  /deep/ .el-rate__text {
    font-size: 1.2rem;
  }

  .rate {
    margin-top: 4rem;
    padding-left: 3rem;
    height: 8rem;
  }
}

}

.line {
  border: 0.5px solid #ddd;
  width: 94%;
  margin: 10px auto;
}
</style>
