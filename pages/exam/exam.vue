<template>
  <view class="container">
    <view v-if="examFinished">
      <view class="summary">
        <text>答题完毕</text>
        <text>正确题数：{{ correctCount }}</text>
        <text>错误题数：{{ wrongCount }}</text>
        <text>{{ correctCount >= 25 ? '考试通过' : '考试失败' }}</text>
        <button @click="restartExam">再来一次</button>
      </view>
    </view>
    <view v-else>
      <!-- 进度条区域 START -->
      <view class="progress-bar-wrapper">
        <view class="progress-bar">
          <view
            class="progress-bar-inner"
            :style="{ width: progressPercent + '%' }"
          ></view>
        </view>
        <text class="progress-text">{{ currentIndex + 1 }}/{{ examQuestions.length }}</text>
      </view>
      <!-- 进度条区域 END -->

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
  </view>
</template>

<script>
export default {
  data() {
    return {
      fullQuestionBank: [],
      examQuestions: [],
      currentIndex: 0,
      currentQuestion: {},
      shuffledOptions: [],
      answered: false,
      selectedIndex: null,
      correct: false,
      correctCount: 0,
      wrongCount: 0,
      examFinished: false,
    };
  },
  computed: {
    // 新增进度百分比计算
    progressPercent() {
      if (this.examQuestions.length === 0) return 0;
      return ((this.currentIndex + 1) / this.examQuestions.length * 100).toFixed(1);
    }
  },
  onLoad() {
    this.loadExamQuestions();
  },
  methods: {
loadExamQuestions() {
  const filePath = uni.getStorageSync('selectedQuestionBank') || '/static/questionsALL.json';

  if (process.env.UNI_PLATFORM === 'h5') {
    uni.request({
      url: filePath,
      success: (res) => {
        this.fullQuestionBank = res.data;
        this.examQuestions = this.shuffleArray(this.fullQuestionBank).slice(0, 30);
        this.currentIndex = 0;
        this.setCurrentQuestion();
        this.correctCount = 0;
        this.wrongCount = 0;
        this.examFinished = false;
      },
      fail: (err) => {
        console.error('加载题库失败(H5)', err);
      }
    });
  } else if (typeof plus !== 'undefined') {
    let relativePath = filePath;
    if (filePath.startsWith('/static')) {
      relativePath = '_www' + filePath;
    }

    plus.io.resolveLocalFileSystemURL(
      relativePath,
      (entry) => {
        entry.file((file) => {
          const reader = new plus.io.FileReader();
          reader.onloadend = (evt) => {
            try {
              const jsonData = JSON.parse(evt.target.result);
              this.fullQuestionBank = jsonData;
              this.examQuestions = this.shuffleArray(this.fullQuestionBank).slice(0, 30);
              this.currentIndex = 0;
              this.setCurrentQuestion();
              this.correctCount = 0;
              this.wrongCount = 0;
              this.examFinished = false;
            } catch (e) {
              console.error('JSON 解析失败', e);
            }
          };
          reader.readAsText(file, 'utf-8');
        });
      },
      (err) => {
        console.error('加载题库失败(APP)', err);
      }
    );
  }
},
    setCurrentQuestion() {
      this.answered = false;
      this.correct = false;
      this.selectedIndex = null;

      const q = this.examQuestions[this.currentIndex];
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

      if (this.correct) {
        this.correctCount++;
        setTimeout(() => {
          this.goNext();
        }, 1000);
      } else {
        this.wrongCount++;
        this.saveWrongQuestion(this.currentQuestion);
        setTimeout(() => {}, 2000);
      }
    },
    saveWrongQuestion(question) {
      const key = 'wrongQuestions';
      uni.getStorage({
        key,
        success: (res) => {
          let existing = res.data || [];
          if (!existing.find(item => item.id === question.id)) {
            existing.push(question);
            uni.setStorage({ key, data: existing });
          }
        },
        fail: () => {
          uni.setStorage({ key, data: [question] });
        }
      });
    },
    nextQuestion() {
      this.goNext();
    },
    goNext() {
      if (this.currentIndex < this.examQuestions.length - 1) {
        this.currentIndex++;
        this.setCurrentQuestion();
      } else {
        this.examFinished = true;
      }
    },
    restartExam() {
      this.loadExamQuestions();
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

/* 进度条样式 */
.progress-bar-wrapper {
  display: flex;
  align-items: center;
  margin-bottom: 12px;
}
.progress-bar {
  flex: 1;
  height: 8px;
  background-color: #eee;
  border-radius: 4px;
  overflow: hidden;
  margin-right: 12px;
}
.progress-bar-inner {
  height: 100%;
  background-color: #2196f3;
  transition: width 0.3s ease;
}
.progress-text {
  font-size: 14px;
  color: #666;
  min-width: 40px;
  text-align: right;
}
.summary text {
  display: block;
  margin: 10px 0;
  font-size: 18px;
}
</style>
