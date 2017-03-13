<template>
  <div class="question">
    <el-row>
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <div class="qst-group" v-for="(item, index) in questionList" v-bind:key="item" v-show="index === current">
            <h3>第{{ index+1 }}题. {{ item.question }}</h3>
            <div class="radio" v-if="item.type === 0">
              <el-radio-group v-model="item.answer">
                <el-radio v-for="child in item.optional" :key="child" :label="child.label">
                  {{ child.option }}
                </el-radio>
              </el-radio-group>
            </div>
            <div class="checkbox" v-if="item.type === 1">
              <el-checkbox-group v-model="item.answer">
                <el-checkbox v-for="child in item.optional" :key="child" :label="child.label">
                  {{ child.option }}
                </el-checkbox>
              </el-checkbox-group>
            </div>
            <div class="input" v-if="item.type === 2">
              <el-input v-model="item.answer" placeholder="请输入内容"></el-input>
            </div>
            <div class="textarea" v-if="item.type === 3">
              <el-input
                type="textarea"
                :rows="2"
                placeholder="请输入内容"
                v-model="item.answer">
              </el-input>
            </div>
            <div class="switch" v-if="item.type === 4">
              <el-switch
                v-model="item.answer"
                on-text=""
                off-text="">
              </el-switch>
            </div>
            <div class="slider" v-if="item.type === 5">
              <el-slider
                v-model="item.answer"
                show-input>
              </el-slider>
            </div>
          </div>
          <div class="qst-group" v-show="lastOpt.submitPage">
            <h1>是否交卷！</h1>
            <el-button type="success" icon="check" @click="result" v-loading.fullscreen.lock="fullscreenLoading">提交</el-button>
            <el-button type="danger" icon="edit" @click="look">查看数据</el-button>
            <div class="result">
              <el-table
                :data="answer"
                stripe
                style="width: 100%">
                <el-table-column
                  prop="question"
                  label="问题">
                </el-table-column>
                <el-table-column
                  prop="answer"
                  label="答案">
                </el-table-column>
              </el-table>
            </div>
          </div>
        </div>
        <div class="result">
          <el-button type="primary" icon="search" @click="goNext" v-if="lastOpt.nextBtn">下一题</el-button>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  data () {
    return {
      current: 0,
      answer: [],
      fullscreenLoading: false,
      lastOpt: {
        nextBtn: true,
        submitPage: false,
      },
      questionList: [
        // type：0为单选题，1为多选题，2为文本输入，3为多行文本输入，4为是否，5为拖动条
        {
          id: 1,
          type: 0,
          mark: 'radio1',
          answer: '',
          question: '请问你对自己目前的英语水平评价为？',
          optional: [
            {
              label: 'a01',
              option: '压根不会'
            }, {
              label: 'a02',
              option: '能读两句'
            }, {
              label: 'a03',
              option: '写英文论文不在话下'
            }, {
              label: 'a04',
              option: '可出任我国外交官'
            }
          ]
        },
        {
          id: 2,
          type: 1,
          mark: 'select2',
          answer: [],
          question: '做这个答题系统的人帅吗？',
          optional: [
            {
              label: 'b01',
              option: '你猜猜'
            }, {
              label: 'b02',
              option: '吹牛逼'
            }, {
              label: 'b03',
              option: '太特么帅了'
            }, {
              label: 'b04',
              option: '啊额'
            }
          ]
        },
        {
          id: 3,
          type: 2,
          mark: 'input15',
          answer: '',
          question: '你每天读书时间有多少？',
        },
        {
          id: 4,
          type: 3,
          mark: 'textarea1',
          question: '你感觉最好的书是什么名字，请写出读后感！',
          answer: ''
        },
        {
          id: 5,
          type: 4,
          mark: 'switch1',
          question: '你每天是否锻炼身体',
          answer: false
        },
        {
          id: 6,
          type: 5,
          mark: 'slider1',
          question: '请选择年龄段',
          answer: 20
        }
      ]
    };
  },
  methods: {
    result: function () {
      let list = this.questionList;
      // 未选择的题选出来
      let unSelect = [];
      list.forEach(function(val, index){
        let iVal = val.answer;
        iVal.length == 0 ? unSelect.push(index + 1) : null;
      });
      if(unSelect.length !== 0){
        // 未选择的题选出来
        let unSelectQuestion = unSelect.join('、');
        this.$message.error('您的第' + unSelectQuestion + '题还没有选择，请选择！');
        this.lastOpt.nextBtn = true;
        this.lastOpt.submitPage = false;
        this.current = unSelect[0] - 1;
      } else {
        // 选择规则都通过，执行的事件
        this.fullscreenLoading = true;
        setTimeout(() => {
          this.fullscreenLoading = false;
        }, 2000);
      }
    },
    goNext: function () {
      let list = this.questionList, cnow = this.current;
      // 当前题目的答案
      let val = list[cnow], cache = false;
      // 下面的三元运算符意思是：如果类型是布尔型，则直接为true，因为选是或者选否都有可能
      // 如果不为布尔型，则为数组或者字符串类型，数组长度若为0则false，字符串若为空也是false，如果用if写法，最简便需要7行
      console.log(typeof val.answer);
      if(typeof val.answer === 'boolean'){
        cache = true;
      } else if (typeof val.answer === 'object') {
        if (val.answer.length > 0){
          cache = true;
        }
      } else {
        if(val.answer){
          cache = true;
        }
      }

      // cache = typeof val.answer === 'boolean' ? true : ( val.answer ? true : false );

      if (!cache) {
        this.$message.error('当前题目未通过，请认真填写，否则无法继续');
        return false;
      }
      
      // 若当前题目index小于等于current的值，代表已是最后一题，此时显示最后一页，并且将下一页按钮删除
      if (cnow >= (list.length-1)) {
        this.lastOpt.nextBtn = false;
        this.lastOpt.submitPage = true;
      }
      this.current++;
    },
    look: function () {
      let list = this.questionList;
      let answer = [];
      list.forEach(function(val, index){
        let opt = {};
        opt.question = val.question
        // 如果当前为单选题
        if (val.type === 0) {
          // 循环当前的所有答案
          for (let i = 0; i < val.optional.length; i++) {
            // 如果对上号了，就打印出来
            if (val.optional[i].label == val.answer)
            {
              opt.answer = val.optional[i].option;
              // console.log(val.optional[i].option);
            }
          }
        }else if (typeof val.answer === 'object') {
          let cache = [];
          for (let i = 0; i < val.answer.length; i++) {
            for (let alli = 0; alli < val.optional.length; alli++) {
              if (val.answer[i] == val.optional[alli].label) {
                cache += val.optional[alli].option + '、'
              }
            }
          }
          opt.answer = cache;
          // console.log(cache);
        } else if (typeof val.answer === 'boolean') {
          var cache = val.answer ? 'yes' : 'no' ;
          opt.answer = cache;
          // console.log(cache);
        } else {
          opt.answer = val.answer;
          // console.log(val.answer);
        }
        answer.push(opt);
      });
      this.answer = answer;
    }
  }
}
</script>

<style>
  .grid-content {
    margin-bottom: 20px;
  }
  .result {
    margin-top: 20px;
  }
</style>