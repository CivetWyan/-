<template>
  <view class="container">
    <view class="title">选择题库等级</view>
    <picker :range="levels" @change="onLevelChange">
      <view class="picker">
        当前选择：{{ levels[selectedIndex] }}
      </view>
    </picker>
    <button @click="onSaveClick">保存</button>

    <!-- 弹窗遮罩 -->
    <view v-if="showConfirm" class="modal-overlay">
      <view class="modal">
        <text class="modal-text">更换题库会导致所有数据都会被清除哦~</text>
        <view class="button-group">
          <button
            :disabled="buttonsDisabled"
            :class="['confirm-btn', buttonsDisabled ? 'disabled-btn' : '']"
            @click="confirmChange"
          >
            确认
          </button>
          <button
            :disabled="buttonsDisabled"
            :class="['cancel-btn', buttonsDisabled ? 'disabled-btn' : '']"
            @click="cancelChange"
          >
            返回
          </button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      levels: ['业余无线电A类', '业余无线电B类', '业余无线电C类', '业余无线电总题库'],
      files: [
        '/static/questionsA.json',
        '/static/questionsB.json',
        '/static/questionsC.json',
        '/static/questionsALL.json'
      ],
      selectedIndex: 0,
      showConfirm: false,
      buttonsDisabled: true,
      previousSelectedFile: ''
    };
  },
  onLoad() {
    this.loadSavedLevel();
  },
  methods: {
    onLevelChange(e) {
      this.selectedIndex = e.detail.value;
    },
    loadSavedLevel() {
      const saved = uni.getStorageSync('selectedQuestionBank');
      if (saved) {
        this.previousSelectedFile = saved;
        const index = this.files.indexOf(saved);
        if (index !== -1) {
          this.selectedIndex = index;
        }
      }
    },
    onSaveClick() {
      const newSelectedFile = this.files[this.selectedIndex];
      if (newSelectedFile !== this.previousSelectedFile) {
        // 显示确认弹窗
        this.showConfirm = true;
        this.buttonsDisabled = true;
        // 2秒后启用按钮
        setTimeout(() => {
          this.buttonsDisabled = false;
        }, 2000);
      } else {
        // 题库未改变，直接保存
        this.saveSelection();
      }
    },
    confirmChange() {
      if (this.buttonsDisabled) return;

      // 清除所有练习数据，保留选中的题库路径
      this.clearAllDataExceptSelected();

      // 保存当前选择
      this.saveSelection();

      // 关闭弹窗
      this.showConfirm = false;
    },
    cancelChange() {
      if (this.buttonsDisabled) return;
      // 关闭弹窗，不做任何操作
      this.showConfirm = false;
    },
    saveSelection() {
      const selectedFile = this.files[this.selectedIndex];
      uni.setStorageSync('selectedQuestionBank', selectedFile);
      this.previousSelectedFile = selectedFile;
      uni.showToast({ title: '已保存', icon: 'success' });
    },
    clearAllDataExceptSelected() {
      // 要保留的key
      const keepKey = 'selectedQuestionBank';

      // 获取所有key
      uni.getStorageInfo({
        success: (res) => {
          const keysToRemove = res.keys.filter(k => k !== keepKey);
          if (keysToRemove.length === 0) return;

          // 批量移除
          keysToRemove.forEach(k => {
            uni.removeStorageSync(k);
          });
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
.title {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 20px;
}
.picker {
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-bottom: 20px;
}
button {
  background-color: #2196f3;
  color: white;
  padding: 12px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
}
/* 弹窗遮罩 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
.modal {
  width: 300px;
  background-color: #fff;
  border-radius: 10px;
  padding: 20px;
  text-align: center;
}
.modal-text {
  font-size: 16px;
  margin-bottom: 20px;
}
.button-group {
  display: flex;
  justify-content: space-around;
}
.confirm-btn {
  background-color: #4caf50; /* 绿色 */
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  flex: 1;
  margin-right: 10px;
  cursor: pointer;
}
.cancel-btn {
  background-color: #f44336; /* 红色 */
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  flex: 1;
  cursor: pointer;
}
.disabled-btn {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>
