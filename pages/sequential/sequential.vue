<template>
  <view class="container">
    <view class="progress-bar-wrapper">
      <view class="progress-bar">
        <view
          class="progress-bar-inner"
          :style="{ width: progressPercent + '%' }"
        ></view>
      </view>
      <text class="progress-text">{{ currentIndex + 1 }}/{{ questions.length }}</text>
    </view>

    <view class="question">{{ currentQuestion.question }}</view>
    <view v-for="(option, index) in shuffledOptions" :key="index">
      <button
        class="option"
        :class="getButtonClass(index)"
        :disabled="answered"
        @click="selectOption(index)"
      >
        {{ option.text }}
      </button>
    </view>

    <view v-if="answered && !correct">
      <button @click="nextQuestion">下一题</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      questions: [],
      currentIndex: 0,
      currentQuestion: {},
      shuffledOptions: [],
      answered: false,
      selectedIndex: null,
      correct: false,
    };
  },
  computed: {
    progressPercent() {
      if (this.questions.length === 0) return 0;
      return ((this.currentIndex + 1) / this.questions.length * 100).toFixed(1);
    }
  },
  onLoad() {
    this.loadProgress();
    this.loadQuestions();
  },
  methods: {
    loadProgress() {
      const savedIndex = uni.getStorageSync('currentIndex');
      if (typeof savedIndex === 'number' && savedIndex >= 0) {
        this.currentIndex = savedIndex;
      }
    },
    saveProgress() {
      uni.setStorageSync('currentIndex', this.currentIndex);
    },
    loadQuestions() {
        const selectedPath =
          uni.getStorageSync("selectedQuestionBank") || "/static/questionsALL.json";
    
        // 平台判断
        if (process.env.UNI_PLATFORM === "h5") {
          // H5 端可以直接使用 URL 请求
          uni.request({
            url: selectedPath,
            success: (res) => {
              this.questions = res.data;
              this.setCurrentQuestion();
            },
            fail: (err) => {
              console.error("加载题库失败(H5)", err);
            },
          });
        } else if (plus) {
          // APP-PLUS 端使用 plus.io + FileReader
          let relativePath = selectedPath;
          if (selectedPath.startsWith("/static")) {
            relativePath = "_www" + selectedPath; // _www/static/xxx.json
          }
    
          plus.io.resolveLocalFileSystemURL(
            relativePath,
            (entry) => {
              entry.file((file) => {
                const reader = new plus.io.FileReader();
                reader.onloadend = (evt) => {
                  try {
                    const jsonData = JSON.parse(evt.target.result);
                    this.questions = jsonData;
                    this.setCurrentQuestion();
                  } catch (e) {
                    console.error("JSON 解析失败", e);
                  }
                };
                reader.readAsText(file, "utf-8");
              });
            },
            (err) => {
              console.error("加载题库失败(APP)", err);
            }
          );
        }
      },
    setCurrentQuestion() {
      this.answered = false;
      this.correct = false;
      this.selectedIndex = null;

      const q = this.questions[this.currentIndex];
      this.currentQuestion = q;

      const options = [
        { text: q.option_A, correct: true },
        { text: q.option_B, correct: false },
        { text: q.option_C, correct: false },
        { text: q.option_D, correct: false },
      ];
      this.shuffledOptions = this.shuffleArray(options);
    },
    shuffleArray(array) {
      return array.sort(() => Math.random() - 0.5);
    },
    selectOption(index) {
      this.answered = true;
      this.selectedIndex = index;
      this.correct = this.shuffledOptions[index].correct;

      if (!this.correct) {
        this.saveWrongQuestion(this.currentQuestion);
        setTimeout(() => {}, 2000);
      } else {
        setTimeout(() => {
          this.goNext();
        }, 1000);
      }
    },
    saveWrongQuestion(question) {
        const key = 'wrongQuestions';
        const toSave = {
          id: question.id,
          question: question.question,
          option_A: question.option_A,
          option_B: question.option_B,
          option_C: question.option_C,
          option_D: question.option_D,
        };
        uni.getStorage({
          key,
          success: res => {
            const arr = res.data || [];
            if (!arr.find(item => item.id === question.id)) {
              arr.push(toSave);
              uni.setStorage({ key, data: arr });
            }
          },
          fail: () => {
            uni.setStorage({ key, data: [toSave] });
          }
        });
      },
    nextQuestion() {
      this.goNext();
    },
    goNext() {
      if (this.currentIndex < this.questions.length - 1) {
        this.currentIndex++;
        this.saveProgress();
        this.setCurrentQuestion();
      } else {
        uni.showToast({ title: '已完成所有题目' });
        uni.removeStorageSync('currentIndex');
      }
    },
    getButtonClass(index) {
      if (!this.answered) return '';
      const opt = this.shuffledOptions[index];
      if (index === this.selectedIndex) {
        return opt.correct ? 'option-correct' : 'option-wrong';
      }
      if (opt.correct) return 'option-correct';
      return 'option-disabled';
    }
  }
};
</script>


<style>
.container {
  padding: 20px;
}
.progress-bar-wrapper {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}
.progress-bar {
  flex: 1;
  height: 10px;
  background-color: #ddd;
  border-radius: 5px;
  overflow: hidden;
  margin-right: 10px;
}
.progress-bar-inner {
  height: 100%;
  background-color: #4caf50;
  transition: width 0.3s ease;
}
.progress-text {
  font-size: 14px;
  color: #333;
}
.question {
  font-size: 18px;
  margin-bottom: 20px;
}
button {
  margin: 10px 0;
  padding: 12px;
  font-size: 16px;
  width: 100%;
  border-radius: 8px;
  border: none;
  color: #000;
}
.option {
  background-color: #eee;
}
.option-correct {
  background-color: #4caf50 !important;
  color: white !important;
}
.option-wrong {
  background-color: #f44336 !important;
  color: white !important;
}
.option-disabled {
  background-color: #ccc !important;
  color: #666 !important;
}
</style>
