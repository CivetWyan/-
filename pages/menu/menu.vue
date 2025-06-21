<template>
  <view class="home-container">
    <view class="title">业余无线电刷题工具</view>
    <view class="menu">
      <button class="menu-button" @click="goToSequential">顺序练题</button>
      <button class="menu-button" @click="goToWrongReview">错题回顾</button>
      <button class="menu-button" @click="goToExam">模拟考试</button>
      <view class="level-button-wrapper">
        <button class="menu-button" @click="goToSelect">
          等级选择
          <text class="selected-level">({{ currentLevel }})</text>
        </button>
      </view>
      <button class="clear-button" @click="showConfirm = true">清除数据</button>
    </view>

    <view v-if="showConfirm" class="confirm-overlay">
      <view class="confirm-box">
        <text class="confirm-text">所有数据都会被清除哦~</text>
        <view class="confirm-buttons">
          <button class="confirm-button return" :disabled="!buttonsEnabled" @click="showConfirm = false">返回</button>
          <button class="confirm-button confirm" :disabled="!buttonsEnabled" @click="clearAllData">确认</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      showConfirm: false,
      buttonsEnabled: false,
      currentLevel: ''
    };
  },
  onShow() {
    this.updateCurrentLevel();
  },
  watch: {
    showConfirm(val) {
      if (val) {
        this.buttonsEnabled = false;
        setTimeout(() => {
          this.buttonsEnabled = true;
        }, 3000);
      }
    }
  },
  methods: {
    goToSequential() {
      uni.navigateTo({ url: '/pages/sequential/sequential' });
    },
    goToWrongReview() {
      uni.navigateTo({ url: '/pages/wrongReview/wrongReview' });
    },
    goToExam() {
      uni.navigateTo({ url: '/pages/exam/exam' });
    },
    goToSelect() {
      uni.navigateTo({ url: '/pages/selectLevel/selectLevel' });
    },
    clearAllData() {
      const keep = uni.getStorageSync('selectedQuestionBank');
      uni.clearStorageSync();
      uni.setStorageSync('selectedQuestionBank', keep);
      this.updateCurrentLevel();
      this.showConfirm = false;
      uni.showToast({ title: '数据已清除', icon: 'none' });
    },
    updateCurrentLevel() {
      const selected = uni.getStorageSync('selectedQuestionBank');
      const map = {
        '/static/questionsA.json': '业余无线电A类',
        '/static/questionsB.json': '业余无线电B类',
        '/static/questionsC.json': '业余无线电C类',
        '/static/questionsALL.json': '业余无线电总题库'
      };
      this.currentLevel = map[selected] || '业余无线电总题库';
    }
  }
};
</script>

<style>
.home-container {
  padding: 40px 20px;
  text-align: center;
}
.title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 40px;
}
.menu-button {
  margin: 12px 0;
  padding: 14px;
  font-size: 18px;
  width: 100%;
  border-radius: 10px;
  background-color: #007aff;
  color: white;
  border: none;
  position: relative;
}
.selected-level {
  font-size: 14px;
  color: #ff0000;
  margin-left: 8px;
}
.clear-button {
  margin: 12px 0;
  padding: 14px;
  font-size: 18px;
  width: 100%;
  border-radius: 10px;
  background-color: #ff0000;
  color: white;
  border: none;
}
.menu-button:disabled {
  background-color: #ccc;
  color: #999;
}
.confirm-overlay {
  position: fixed;
  top: 0; left: 0;
  right: 0; bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}
.confirm-box {
  background: white;
  padding: 24px;
  border-radius: 10px;
  width: 80%;
  max-width: 300px;
  text-align: center;
}
.confirm-text {
  font-size: 16px;
  margin-bottom: 20px;
  display: block;
}
.confirm-buttons {
  display: flex;
  justify-content: space-between;
}
.confirm-button {
  flex: 1;
  margin: 0 6px;
  padding: 10px;
  font-size: 16px;
  border-radius: 6px;
  color: white;
  border: none;
}
.confirm-button.return {
  background-color: #f44336;
}
.confirm-button.confirm {
  background-color: #4caf50;
}
.confirm-button:disabled {
  opacity: 0.6;
}
</style>
