<template>
  <view class="container">
    <view class="progress-bar-wrapper" v-if="questions.length">
      <view class="progress-bar">
        <view
          class="progress-bar-inner"
          :style="{ width: progressPercent + '%' }"
        ></view>
      </view>
      <text class="progress-text">{{ answeredCount }}/{{ totalCount }}</text>
    </view>

    <view v-if="currentQuestion" class="question">{{ currentQuestion.question }}</view>
    <view v-for="(option, idx) in shuffledOptions" :key="idx">
      <button
        class="option"
        :class="getButtonClass(idx)"
        :disabled="answered"
        @click="selectOption(idx)"
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
      currentQuestion: null,
      shuffledOptions: [],
      answered: false,
      selectedIndex: null,
      correct: false,
      totalCount: 0,
      answeredCount: 0,
      removeQueue: [],
    };
  },
  computed: {
    progressPercent() {
      return this.totalCount
        ? ((this.answeredCount / this.totalCount) * 100).toFixed(1)
        : 0;
    },
  },
  onLoad() {
    this.loadWrongQuestions();
  },
  methods: {
    loadWrongQuestions() {
      uni.getStorage({
        key: 'wrongQuestions',
        success: (res) => {
          const arr = res.data || [];
          if (arr.length) {
            this.questions = JSON.parse(JSON.stringify(arr));
            this.totalCount = this.questions.length;
            this.setCurrentQuestion();
          } else {
            uni.showToast({ title: '暂无错题', icon: 'none' });
          }
        },
        fail: () => {
          uni.showToast({ title: '暂无错题记录', icon: 'none' });
        }
      });
    },
    setCurrentQuestion() {
      if (this.currentIndex >= this.questions.length) {
        uni.showToast({ title: '错题回顾完成' });
        return;
      }

      this.answered = false;
      this.correct = false;
      this.selectedIndex = null;
      this.currentQuestion = this.questions[this.currentIndex];
      const opts = [
        { text: this.currentQuestion.option_A, correct: true },
        { text: this.currentQuestion.option_B, correct: false },
        { text: this.currentQuestion.option_C, correct: false },
        { text: this.currentQuestion.option_D, correct: false }
      ];
      this.shuffledOptions = this.shuffleArray(opts);
    },
    shuffleArray(arr) {
      return arr.sort(() => 0.5 - Math.random());
    },
    selectOption(idx) {
      this.answered = true;
      this.selectedIndex = idx;
      this.correct = this.shuffledOptions[idx].correct;

      if (!this.correct) {
        // 答错：等待1秒，用户手动点击下一题
        this.saveWrongQuestion(this.currentQuestion);
      } else {
        // 答对：1秒后自动进入下一题
        this.removeQueue.push(this.questions[this.currentIndex].id);
        setTimeout(() => {
          this.answeredCount++;
          this.nextQuestion();
        }, 1000);
      }
    },
    nextQuestion() {
      if (this.correct === false) {
        this.answeredCount++;
      }
      this.currentIndex++;
      this.setCurrentQuestion();
    },
    getButtonClass(idx) {
      if (!this.answered) return '';
      const opt = this.shuffledOptions[idx];
      if (idx === this.selectedIndex) {
        return opt.correct ? 'option-correct' : 'option-wrong';
      }
      return opt.correct ? 'option-correct' : 'option-disabled';
    },
    saveWrongQuestion(question) {
      // 保存错题（去重）
      uni.getStorage({
        key: 'wrongQuestions',
        success: (res) => {
          let list = res.data || [];
          const exists = list.some(q => q.id === question.id);
          if (!exists) {
            list.push(question);
            uni.setStorage({ key: 'wrongQuestions', data: list });
          }
        },
        fail: () => {
          uni.setStorage({ key: 'wrongQuestions', data: [question] });
        }
      });
    }
  },
  onUnload() {
    if (this.removeQueue.length > 0) {
      uni.getStorage({
        key: 'wrongQuestions',
        success: (res) => {
          let updated = (res.data || []).filter(
            q => !this.removeQueue.includes(q.id)
          );
          uni.setStorage({ key: 'wrongQuestions', data: updated });
        }
      });
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