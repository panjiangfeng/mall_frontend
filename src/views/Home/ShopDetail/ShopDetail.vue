<template>
<div id="parent">
  <div class="dtitle">Test Shop<br/>Do Yourself</div>
  <h2>
     Test Shop❤️
</h2>
  <div class="background" id="child1">
    <span @click="thanksskim">Shop</span>
  </div>
<!--  引入遮罩-->
  <zhezhao></zhezhao>
</div>
</template>
<script>
import zhezhao from "@/components/common/zhezhao";
export default {
  // beforeCreate() {
  //   document.querySelector('body').setAttribute('style', "margin: 0;padding: 0;height: 200vh;overflow-x: hidden;")
  //   document.querySelector('html').setAttribute('style', 'margin: 0;padding: 0;height: 200vh;overflow-x: hidden;')
  //
  //
  // },
  // beforeDestroy() {
  //   document.querySelector('body').removeAttribute('style')
  //   document.querySelector('html').removeAttribute('style')
  //
  // },
  components:{
    zhezhao
  },
  mounted(){
    window.addEventListener('scroll',this.scroll) // 监听滚动事件
  },
  methods:{
    scroll(){
      const background = document.querySelector(".background")
      document.addEventListener('scroll', () => {
        const scrollY = window.scrollY

        if (scrollY !== 0) {
          background.style.backgroundPosition = `calc(50% + ${scrollY}px) calc(50% + ${scrollY}px)`
        }else{
          background.style.backgroundPosition = ''
        }
      })
    },
    thanksskim(){
      this.$message.success("Thanks browsing~ ❤️")
    }
  },
}
</script>

<style scoped>
.background {
  background-image: url("~assets/img/mountain range.jpg");
  background-size: cover;
  background-position: 50% 50%;
  height: 200vh;
  font-weight: 900;
  font-size: 25rem;
  line-height: 130vh;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  position: relative;
  text-align: center;
  overflow: hidden;
}

.background::before {
  content: '';
  background-size: cover;
  background-image: inherit;
  background-position: 50% 50%;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  z-index: -99;
}

.dtitle {
  margin-top: 10vh;
  font-size: 5rem;
  font-weight: 500;
  position: absolute;
  text-align: center;
  width: 100%;
  letter-spacing: 10px;
  color: rgba(168, 252, 222, 0.5);
  font-family: UbuntuMono-Bold;

}

h2 {
  font-family: UbuntuMono-Bold;
  position: absolute;
  letter-spacing: 2px;
  top: 120vh;
  width: 60%;
  color: #fff;
  left: 50%;
  transform: translateX(-50%);
  padding: 30px;
  background-color: rgba(0, 0, 0, .3);
}
</style>
