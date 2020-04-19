<template>
  <div id="app">
    <transition name="slide-fade">
        <div ref="toast" v-show="toastShow" class="toast">{{this.message}}</div>
      </transition>
    <div class="outer" v-if="boxShow">
      <div class="inputBox">
        请输入问卷编号
        <br />
        <input type="text" v-model="id" class="outerInput" />
        <br />
        <br />
        <div class="btn" @click="handlerGetQuestionnaire">确认</div>
      </div>
    </div>
    <div class="inner" v-if="!boxShow">
      <div class="title">{{questionnaireData.name}}</div>
      <div class="question-box">
        <el-form label-width="80px">
          <el-form-item
            v-for="(item,index) in questionnaireData.questions"
            :key="index"
            :label="'第'+(index+1)+'题:'"
          >
            <template v-if="item.questionType==1">
              {{item.name}}
              <br />
              <el-radio-group v-model="answer[index]">
                <el-radio
                  v-for="(item1,index1) in item.options"
                  :key="index1+'1'"
                  :label="index1"
                >{{item1.content}}</el-radio>
              </el-radio-group>
            </template>
            <template v-if="item.questionType==2">
              {{item.name}}
              <br />
              <el-checkbox-group v-model="answer[index]">
                <el-checkbox
                  v-for="(item1,index1) in item.options"
                  :key="index1+'2'"
                  :label="index1"
                >{{item1.content}}</el-checkbox>
              </el-checkbox-group>
            </template>
            <template v-if="item.questionType==3">
              {{item.name}}
              <br />
              <el-input type="textarea" :rows="3" v-model="answer[index]"></el-input>
            </template>
          </el-form-item>
        </el-form>
        <el-button class="btn1" type="primary" @click="handlerSubmit">上传问卷</el-button>
      </div>
    </div>
  </div>
</template>

<script>
import { get,post_json } from "./http/utils";

export default {
  name: "App",
  data() {
    return {
      answer: [],
      id: "",
      boxShow: true,
      toastShow: false,
      message: "测试版",
      questionnaireData: []
    };
  },
  methods: {
    toast(message, type) {
      this.toastShow = true;
      setTimeout(() => {
        this.toastShow = false;
      }, 2000);
      this.message = message;
      if (type === "error") {
        this.$refs.toast.style.background = "#F56C6C";
      }
    },
    handlerGetQuestionnaire() {
      if (this.id === "") {
        this.toast("您未输入问卷编号", "error");
        return;
      }
      get("/api/question/survey/" + this.id)
        .then(res => {
          if (res.data.code == 200) {
            this.questionnaireData = res.data.data;
            this.questionnaireData.questions.forEach((item, index) => {
              if (item.questionType == 2) {
                this.answer[index] = [];
              }
            });
            this.toast("获取问卷成功");
            this.boxShow = false;
          } else {
            this.toast("未获取问卷", "error");
          }
        })
        .catch(() => {
          this.toast("未获取问卷", "error");
        });
    },
    handlerSubmit() {
      let arr = [];
      console.log(this.questionnaireData)
      this.questionnaireData.questions.forEach((item,index) => {
        arr.push({
          questionId: item.id,
          answer:this.answer[index] instanceof Array?this.answer[index].join(''):this.answer[index]
        });
      });
      console.log(arr)
      post_json('/api/question/answer',{
        id:this.questionnaireData.id,
        answers:arr
      }).then((res)=>{
        console.log(res.data.code, '---------')
        if(res.data.code==200){
          console.log(12341)
          console.log(this)
          this.toast("上传成功");
          this.boxShow = true
        }else{
           this.toast('上传失败', "error");
        }
      })
    }
  }
};
</script>

<style>
.title {
  font-size: 28px;
  margin-bottom: 10px;
}
.question-box {
  text-align: left;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.inputBox {
  margin: 10% auto;
}
.outerInput {
  width: 80%;
  height: 40px;
}
.btn {
  display: inline-block;
  width: 100px;
  height: 60px;
  line-height: 60px;
  color: white;
  background: #409eff;
  border-radius: 5px;
}
.toast {
  position: fixed;
  top: 10%;
  left: 50%;
  transform: translateX(-50%);
  padding: 15px;
  border-radius: 5px;
  width: 80%;
  max-width: 400px;
  color: white;
  background: #67c23a;
}
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
  opacity: 0;
}
.inner {
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
  padding: 10px;
  max-width: 800px;
}
@media only screen and (min-width: 1029px) {
  .inner {
    border: 1px solid #ededed;
    border-radius: 5px;
  }
}
.btn1 {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 10px;
}
</style>
