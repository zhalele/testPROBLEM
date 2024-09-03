<template>
  <div class="mb-4">
    <h3 style="text-align: center;">安全题目测试</h3>
  </div>
  <div v-if="list && !isEnd">
    <!-- <p> {{ count }}</p> -->
    <el-alert :title="`题目总数为${total}道题，每次随机抽取50道题`" type="info" :closable="false" />
    <div class="sub-title">当前题目: <span class="weight">{{ count + 1 }}/{{ list.length }}</span> </div>
    <div style="display: flex; align-items: center; justify-content: center;">
      <el-button size="large" type="success" round @click="changeO" :icon="ArrowLeft">上一题</el-button>
      <el-button size="large" type="success" round @click="changeB" :icon="ArrowRight">下一题</el-button>
    </div>
    <p style="font-weight: bold;">{{ count + 1 }}、{{ list[count].title }}</p>
    <div style="margin: 10px 0 ;">已选择的答案：<span class="weight"> {{ list[count].option.answer }}</span></div>
    <!-- <el-alert :title="`正确答案：${list[count].trueAnswer}`" type="success" :closable="false" /> -->
    <el-radio-group v-model="list[count].option.answer"
      style="display: flex; flex-flow: column nowrap; align-items: flex-start; width: 100%;">
      <div v-for="i in list[count].option.name">
        <el-radio size="large" label="" :value="i.split('.')[0]">
          <div class="radio-label ">
            {{ i }}</div>
        </el-radio>
      </div>

    </el-radio-group>
    <div><el-button size="default" type="info" style="display: block; margin: 0 auto;" @click="submit">提交试卷</el-button>
    </div>

  </div>
  <div v-if="isEnd">
    <div class="end-style">
      最终分数<div style="color: #67c23a; text-align: center; font-size: 50px; font-weight: bold;">{{ endScore }}</div>
    </div>
    <div class="end-style" style="margin: 5px 0;">
      以下为错题记录
    </div>
    <div v-if="errList">
      <div v-for="item in errList" class="box">
        <p style="font-weight: bold;">{{ item.title }}</p>
        <el-alert :title="`你选择的答案：${item.option.answer}`" type="error" :closable="false" />
        <div style="height: 5px;"></div>
        <el-alert :title="`正确答案：${item.trueAnswer}`" type="success" :closable="false" />
        <el-radio-group style="display: flex; flex-flow: column nowrap; align-items: flex-start; width: 100%;"
          v-model="item.option.answer" disabled>
          <div v-for="i in item.option.name">
            <div><el-radio size="large" :value="i.split('.')[0]">
                <div style=" font-size: 18px;">{{ i }} </div>
              </el-radio></div>
          </div>

        </el-radio-group>
      </div>
    </div>
  </div>

</template>


<script setup lang="ts">
import { ref, onMounted } from "vue";
import { ElMessage } from "element-plus";
import { List } from './problem'
import {
  ArrowLeft,
  ArrowRight,
  Check
} from '@element-plus/icons-vue'
defineProps<{ msg: string }>();

const count = ref(0);
const total = ref(0);
const list = ref();
const errList = ref(); //错题记录
// 是否已经结束答题
const isEnd = ref(false);
const endScore = ref(0);


const toast = (item: any) => {
  console.log(item)
  console.log(item)

  if (item.option.answer === item.trueAnswer) {
    ElMessage.success("答案正确");
  } else {
    ElMessage.error("答案错误");

  }

  // ElMessage.success("Hello");
};
onMounted(() => {
  errList.value = []
  const a = List.map((item: any) => {
    let optionParts = item.option.split("|");
    return {
      ...item,
      option: {
        name: optionParts, // 将拆分后的数组赋值给 name
        answer: '' // 假设正确答案是数组的第一个元素
      }
    };
  });
  console.log(a)
  const questionsCopy = [...a];
  total.value = a.length
  // 抽取题目 
  pickRandomQuestions(questionsCopy, 50)
  // console.log(list.value)
});


const pickRandomQuestions = (questions: any[], count: number) => {
  const picked = []; // 存储随机选取的题目
  while (count > picked.length) {
    // 随机选择一个索引
    const index = Math.floor(Math.random() * questions.length);
    // 将选中的题目添加到结果数组
    picked.push(questions[index]);
    // 从原始数组中移除选中的题目，避免重复
    questions.splice(index, 1);
  }
  console.log(picked)
  list.value = picked
};


function changeO() {
  if (count.value === 0) {
    return
  }
  count.value--

}
function changeB() {
  if (count.value === list.value.length - 1) {
    return
  }
  if (list.value[count.value].option.answer == '') {
    ElMessage.warning("请先回答当前题目");
    return
  }
  count.value++

}
function submit() {
  let unanswered: any[] = [];
  list.value.forEach((question: any, index: any) => {
    if (!question.option.answer || question.option.answer.trim() === "") {
      // 如果答案未定义、为空或仅包含空白字符，则认为该题目未答  
      unanswered.push(index + 1); // 题目编号从1开始，所以加1  
    }
  });
  if (unanswered.length > 0) {
    // 如果有未答的题目，显示提示信息  
    console.log("有题目未答，具体题目编号如下：", unanswered.join(", "));
    ElMessage.error(`有题目未答,具体题目编号如下: ${unanswered.join(", ")}`);
  } else {

    console.log("所有题目都已回答。");
    checkAnswers()
  }
}

function checkAnswers() {
  const errors: any[] = [];
  let score = 0;
  list.value.forEach((question: any) => {
    if (question.option.answer !== question.trueAnswer) {
      // 如果答案不一致，将错误信息存储到errors数组中  
      errors.push(question);
    } else {
      score++;
    }
  });
  isEnd.value = true
  endScore.value = score
  errList.value = errors
  console.log(errors)
  ElMessage.success(`你的最终分数：${score}`);
}

</script>


<style>
.box {
  /* font-size: 18px; */
}

.weight {
  font-weight: bold;
}

.ep-radio-group {
  line-height: inherit;

}

.ep-radio.ep-radio--large {
  height: auto;
  min-height: 65px;
}

.ep-radio__label {
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: normal;
  word-break: break-all;
}

.radio-label {
  font-size: 15px;
}

.sub-title {
  text-align: center;
  margin: 6px 0;
}

.end-style {
  text-align: center;
  font-size: 20px;
  font-weight: bold;
}
</style>
