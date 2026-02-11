<template>
  <!-- 隐藏的音频标签：打字机音效 -->
  <audio id="typewriterSound" preload="auto">
    <source src="https://assets.mixkit.co/sfx/preview/mixkit-typewriter-enter-2863.mp3" type="audio/mpeg">
   </audio>
  
  <!-- 背景音乐 -->
  <audio id="backgroundMusic" preload="auto" loop>
    <source src="/music/waterfall.mp3" type="audio/mpeg">
   </audio>

  <!-- 音乐控制按钮 -->
  <div class="music-control">
    <button class="music-btn" @click="toggleMusic">
      {{ isMusicPlaying ? '🔊' : '🔇' }}
    </button>
  </div>

  <!-- 开场文案层 -->
  <div v-if="showOpening" class="intro-overlay" id="introOverlay">
    <div class="intro-text" id="introText">
      <div v-for="(line, index) in openingLines" :key="index" :class="['intro-line', { 'active': animatedLines.includes(index) }]">
        {{ line }}
      </div>
    </div>
  </div>
  
  <!-- 清单内容 -->
  <div class="container" :class="{ 'show': !showOpening }">
    <h1>人生已完成清单</h1>
    <div class="status-progress-achievement">
      <div class="completion-status">
        已完成: {{ completedCount }} / {{ totalCount }}
      </div>
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
      </div>
      <div class="achievement-container-inline">
        <button class="achievement-button" @click="toggleAchievementPanel">
          成就 ({{ unlockedAchievementsCount }})
        </button>
        <div v-if="showAchievementPanel" class="achievement-panel">
          <h3>我的成就</h3>
          <div class="achievement-list">
            <div 
              v-for="achievement in achievements" 
              :key="achievement.id"
              :class="['achievement-item', achievement.completed ? 'unlocked' : 'locked']"
            >
              <span class="achievement-icon">{{ achievement.completed ? '🏆' : '🔒' }}</span>
              <div class="achievement-info">
                <h4>{{ achievement.name }}</h4>
                <p>{{ achievement.description }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="button-container">
      <button class="reset-button" @click="resetTags">重选</button>
      <button class="challenge-button" @click="randomChallenge">随机挑战</button>
    </div>
    <div v-if="currentChallenge" class="challenge提示">
      🎯 随机挑战：{{ currentChallenge.text }}
    </div>
    <div v-if="show安慰" class="安慰提示">
      {{ 安慰话语 }}
    </div>
    <div v-if="isEmpty && !show安慰" class="status提示">
      开始标记你的人生清单吧✨
    </div>
    <div v-if="isAllCompleted && !show安慰" class="status提示">
      恭喜！你的人生体验已拉满🎊
    </div>
    <div class="tags-container">
      <div
        v-for="tag in filteredTags"
        :key="tag.id"
        :class="['tag', tag.completed ? 'completed' : 'not-completed']"
        @click="toggleTag(tag.id)"
        @dblclick="startEdit(tag.id)"
      >
        <span v-if="editingId !== tag.id">{{ tag.text }}</span>
        <input 
          v-else 
          type="text" 
          v-model="editText"
          class="edit-input"
          @blur="finishEdit"
          @keyup.enter="finishEdit"
          @keyup.esc="cancelEdit"
          :ref="`editInput_${tag.id}`"
        >
        <button class="delete-btn" @click.stop="deleteTag(tag.id)">×</button>
      </div>
    </div>
    <div class="filter-bar">
      <button 
        class="filter-btn" 
        :class="{ active: filter === 'all' }"
        @click="filter = 'all'"
      >
        全部
      </button>
      <button 
        class="filter-btn" 
        :class="{ active: filter === 'done' }"
        @click="filter = 'done'"
      >
        已完成
      </button>
      <button 
        class="filter-btn" 
        :class="{ active: filter === 'undone' }"
        @click="filter = 'undone'"
      >
        未完成
      </button>
    </div>
    <div class="add-tag-container">
      <input 
        type="text" 
        v-model="newTagText" 
        placeholder="添加新标签"
        class="add-tag-input"
        @keyup.enter="addTag"
      >
      <button class="add-tag-btn" @click="addTag">添加</button>
    </div>
    <div class="theme-container">
      <span>主题：</span>
      <button 
        class="theme-button" 
        :class="{ active: theme === 'light' }"
        @click="changeTheme('light')"
      >
        浅色
      </button>
      <button 
        class="theme-button" 
        :class="{ active: theme === 'dark' }"
        @click="changeTheme('dark')"
      >
        深色
      </button>
    </div>
    <div class="share-container">
      <button class="share-button" @click="shareList">生成截图</button>
      <!-- <button class="share-button" @click="generateShareLink">复制分享链接</button> -->
    </div>
  </div>
  <div v-if="showAchievementPopup" class="achievement-popup">
    <div class="achievement-popup-content">
      <h3>🎉 成就解锁！</h3>
      <h2>{{ currentAchievement.name }}</h2>
      <p>{{ currentAchievement.description }}</p>
    </div>
  </div>
</template>

<script>
import html2canvas from 'html2canvas';
export default {
  name: 'App',
  data() {
    return {
      tags: [
        { id: 1, text: '送礼物', completed: false },
        { id: 2, text: '被送礼物', completed: false },
        { id: 3, text: '暗恋', completed: false },
        { id: 4, text: '明恋', completed: false },
        { id: 5, text: '失恋', completed: false },
        { id: 6, text: '表白', completed: false },
        { id: 7, text: '被表白', completed: false },
        { id: 8, text: '留长发', completed: false },
        { id: 9, text: '剪短发', completed: false },
        { id: 10, text: '染发', completed: false },
        { id: 11, text: '漂发', completed: false },
        { id: 12, text: '烫发', completed: false },
        { id: 13, text: '化妆', completed: false },
        { id: 14, text: '做美甲', completed: false },
        { id: 15, text: '放下一个人', completed: false },
        { id: 16, text: '有过遗憾', completed: false },
        { id: 17, text: '爱而不得', completed: false },
        { id: 18, text: '双向奔赴', completed: false },
        { id: 19, text: '当海王', completed: false },
        { id: 20, text: '拒绝他人表白', completed: false },
        { id: 21, text: '表白被拒', completed: false },
        { id: 22, text: '被渣', completed: false },
        { id: 23, text: '犯过傻', completed: false },
        { id: 24, text: '装糊涂', completed: false },
        { id: 25, text: '犯校规', completed: false },
        { id: 26, text: '打架', completed: false },
        { id: 27, text: '迟到', completed: false },
        { id: 28, text: '旷课', completed: false },
        { id: 29, text: '上课睡觉', completed: false },
        { id: 30, text: '打架被叫家长', completed: false },
        { id: 31, text: '喝酒', completed: false },
        { id: 32, text: '抽烟', completed: false },
        { id: 33, text: '纹身', completed: false },
        { id: 34, text: '去清吧', completed: false },
        { id: 35, text: '和朋友去KTV', completed: false },
        { id: 36, text: '断片', completed: false },
        { id: 37, text: '失眠', completed: false },
        { id: 38, text: '睡一天', completed: false },
        { id: 39, text: '吵架', completed: false },
        { id: 40, text: '绝交', completed: false },
        { id: 41, text: '晚上一个人哭', completed: false },
        { id: 42, text: '捐血', completed: false },
        { id: 43, text: '住院', completed: false },
        { id: 44, text: '做手术', completed: false },
        { id: 45, text: '晕倒', completed: false },
        { id: 46, text: '会做饭', completed: false },
        { id: 47, text: '做一桌菜', completed: false },
        { id: 48, text: '做饭给家人', completed: false },
        { id: 49, text: '做甜品给喜欢的人', completed: false },
        { id: 50, text: '有超过10年的好朋友', completed: false },
        { id: 51, text: '有个无条件可信任的朋友', completed: false },
        { id: 52, text: '买花', completed: false },
        { id: 53, text: '被送花', completed: false },
        { id: 54, text: '给自己买礼物', completed: false },
        { id: 55, text: '通宵补作业', completed: false },
        { id: 56, text: '一个人散步', completed: false },
        { id: 57, text: '夜跑', completed: false },
        { id: 58, text: '深夜散心', completed: false },
        { id: 59, text: '向陌生人吐露心声', completed: false },
        { id: 60, text: '一个人出去吃饭', completed: false },
        { id: 61, text: '一个人看电影', completed: false },
        { id: 62, text: '摄影', completed: false },
        { id: 63, text: '一个人去酒吧', completed: false },
        { id: 64, text: '一个人过生日', completed: false },
        { id: 65, text: '一个人逛超市', completed: false },
        { id: 66, text: '一个人去图书馆', completed: false },
        { id: 67, text: '一个人看病', completed: false },
        { id: 68, text: '一个人去唱歌', completed: false },
        { id: 69, text: '社死过', completed: false },
        { id: 70, text: '一个人出门逛', completed: false },
        { id: 71, text: '一个人在外难过', completed: false },
        { id: 72, text: '给自己写信', completed: false },
        { id: 73, text: '出国', completed: false },
        { id: 74, text: '一个人旅游', completed: false },
        { id: 75, text: '跟朋友旅游', completed: false },
        { id: 76, text: '拥有要好的异性朋友', completed: false },
        { id: 77, text: '谈恋爱', completed: false },
        { id: 78, text: '考试不及格', completed: false },
        { id: 79, text: '考试第一名', completed: false },
        { id: 80, text: '当班干部', completed: false },
        { id: 81, text: '竞选学生会', completed: false },
        { id: 82, text: '上电视', completed: false },
        { id: 83, text: '上报纸', completed: false },
        { id: 84, text: '登台演出', completed: false },
        { id: 85, text: '主持节目', completed: false },
        { id: 86, text: '演讲', completed: false },
        { id: 87, text: '野性消费', completed: false },
        { id: 88, text: '买东西被宰', completed: false },
        { id: 89, text: '被老师点名表扬', completed: false },
        { id: 90, text: '被老师点名批评', completed: false },
        { id: 91, text: '全校表扬', completed: false },
        { id: 92, text: '被背叛', completed: false },
        { id: 93, text: '被伤害', completed: false },
        { id: 94, text: '被坚定选择', completed: false },
        { id: 95, text: '获奖', completed: false },
        { id: 96, text: '学一种语言', completed: false },
        { id: 97, text: '写论文', completed: false },
        { id: 98, text: '写书', completed: false },
        { id: 99, text: '写诗', completed: false },
        { id: 100, text: '写日记', completed: false },
        { id: 101, text: '写剧本', completed: false },
        { id: 102, text: '写歌', completed: false },
        { id: 103, text: '拍影片', completed: false },
        { id: 104, text: '参加比赛', completed: false },
        { id: 105, text: '拍写真', completed: false },
        { id: 106, text: '买相机', completed: false },
        { id: 107, text: '会一种乐器', completed: false },
        { id: 108, text: '有超过5年的兴趣爱好', completed: false },
        { id: 109, text: '参加志愿活动', completed: false },
        { id: 110, text: '自己一个人在外面住过', completed: false },
        { id: 111, text: '看鬼片', completed: false },
        { id: 112, text: '去密室', completed: false },
        { id: 113, text: '去鬼屋', completed: false },
        { id: 114, text: '去游乐场', completed: false },
        { id: 115, text: '去看现场演唱会', completed: false },
        { id: 116, text: '去音乐节', completed: false },
        { id: 117, text: '偶遇明星', completed: false },
        { id: 118, text: '去签售会', completed: false },
        { id: 119, text: '在图书馆待一天', completed: false },
        { id: 120, text: '兼职', completed: false },
        { id: 121, text: '打工', completed: false },
        { id: 122, text: '看画展', completed: false },
        { id: 123, text: '捐款', completed: false },
        { id: 124, text: '道歉', completed: false },
        { id: 125, text: '释怀', completed: false },
        { id: 126, text: '失望', completed: false },
        { id: 127, text: '淋雨', completed: false },
        { id: 128, text: '种花', completed: false },
        { id: 129, text: '养宠物', completed: false },
        { id: 130, text: '泡温泉', completed: false },
        { id: 131, text: '跳伞', completed: false },
        { id: 132, text: '坐过山车', completed: false },
        { id: 133, text: '蹦极', completed: false },
        { id: 134, text: '骑马', completed: false },
        { id: 135, text: '卡丁车', completed: false },
        { id: 136, text: '攀岩', completed: false },
        { id: 137, text: '游泳', completed: false },
        { id: 138, text: '滑雪', completed: false },
        { id: 139, text: '滑冰', completed: false },
        { id: 140, text: '旱冰', completed: false },
        { id: 141, text: '滑板', completed: false },
        { id: 142, text: '去野炊', completed: false },
        { id: 143, text: '登山', completed: false },
        { id: 144, text: '看雪', completed: false },
        { id: 145, text: '看海', completed: false },
        { id: 146, text: '看日出', completed: false },
        { id: 147, text: '看日落', completed: false },
        { id: 148, text: '拿驾照', completed: false },
        { id: 149, text: '自驾游', completed: false },
        { id: 150, text: '给父母买衣服', completed: false },
        { id: 151, text: '当伴娘/郎', completed: false },
        { id: 152, text: '摆地摊', completed: false },
        { id: 153, text: '和同学打水仗', completed: false },
        { id: 154, text: '社团活动', completed: false },
        { id: 155, text: '组建社团', completed: false },
        { id: 156, text: '写信', completed: false },
        { id: 157, text: '独自坐飞机', completed: false },
        { id: 158, text: '拼拼图', completed: false },
        { id: 159, text: '办生日会', completed: false },
        { id: 160, text: '健身', completed: false },
        { id: 161, text: '买名牌', completed: false },
        { id: 162, text: '开网店', completed: false },
        { id: 163, text: '练字', completed: false },
        { id: 164, text: '沉迷游戏', completed: false },
        { id: 165, text: '拍毕业照', completed: false },
        { id: 166, text: '面试成功', completed: false },
        { id: 167, text: '放孔明灯', completed: false },
        { id: 168, text: '逛小吃街', completed: false },
        { id: 169, text: '和网友见面', completed: false },
        { id: 170, text: '发表成果', completed: false },
        { id: 171, text: '买唱片', completed: false },
        { id: 172, text: '异地恋', completed: false },
        { id: 173, text: '异国恋', completed: false },
        { id: 174, text: '拍写真', completed: false },
        { id: 175, text: '认识不同国籍的人', completed: false },
        { id: 176, text: '创业', completed: false },
        { id: 177, text: '看极光', completed: false },
        { id: 178, text: '进公司工作', completed: false },
        { id: 179, text: '做未来规划', completed: false },
        { id: 180, text: '搬家', completed: false },
        { id: 181, text: '转学', completed: false },
        { id: 182, text: '被骗', completed: false },
        { id: 183, text: '撒谎', completed: false },
        { id: 184, text: '成功', completed: false },
        { id: 185, text: '改掉一个坏习惯', completed: false },
        { id: 186, text: '加入校队', completed: false },
        { id: 187, text: '坐摩天轮', completed: false },
        { id: 188, text: '在朋友家过夜', completed: false },
        { id: 189, text: '擅长一项运动', completed: false }
      ],
      show安慰: false,
      安慰话语: '没关系，人生就是一场不断体验的旅程，慢慢来，你已经很棒了！',
      filter: 'all',
      newTagText: '',
      editingId: -1,
      editText: '',
      theme: 'light',
      achievements: [
        { id: 1, name: '人生体验家', description: '完成10项人生体验', completed: false, requirement: { type: 'count', value: 10 } },
        { id: 2, name: '独处大师', description: '完成一个人旅游、看电影和吃饭', completed: false, requirement: { type: 'tags', values: ['一个人旅游', '一个人看电影', '一个人出去吃饭'] } },
        { id: 3, name: '社交达人', description: '完成5项社交相关体验', completed: false, requirement: { type: 'category', value: 'social', count: 5 } },
        { id: 4, name: '冒险家', description: '完成3项冒险活动', completed: false, requirement: { type: 'category', value: 'adventure', count: 3 } }
      ],
      showAchievementPopup: false,
      currentAchievement: null,
      showAchievementPanel: false,
      currentChallenge: null,
      showOpening: true,
      isMusicPlaying: false,
      openingLines: [
        '人生是旷野，不是轨道。',
        '',
        '所以这里没有你必须抵达的站台，没有定好的发车时间，也没有谁规定你得沿着铁轨一路狂奔。',
        '',
        '只有你，和这片望不到边的、等着被你认识的自己。',
        '',
        '风从哪个方向来，你就往哪个方向走一走。',
        '累了就坐下，看见野花就蹲下来看看。',
        '想往深处去，那就去——反正路都是你踩出来的。',
        '',
        '这张清单，不是地图，更不是行军令。',
        '它只是一只挎在肩上的布袋子，帮你一路走，一路拾起那些真正打动你的东西。',
        '',
        '不着急。',
        '旷野又不会跑。'
      ],
      animatedLines: []
    }
  },
  mounted() {
    this.loadFromLocalStorage()
    this.loadTheme()
    this.loadAchievements()
    this.checkAchievements()
    this.loadFromShareLink()
    this.startOpeningAnimation()
    // 初始化并播放背景音乐
    this.initMusic()
  },
  computed: {
    totalCount() {
      return this.tags.length
    },
    completedCount() {
      return this.tags.filter(tag => tag.completed).length
    },
    progressPercentage() {
      return (this.completedCount / this.totalCount * 100).toFixed(0)
    },
    isEmpty() {
      return this.completedCount === 0
    },
    isAllCompleted() {
      return this.completedCount === this.totalCount
    },
    filteredTags() {
      if (this.filter === 'all') {
        return this.tags
      } else if (this.filter === 'done') {
        return this.tags.filter(tag => tag.completed)
      } else if (this.filter === 'undone') {
        return this.tags.filter(tag => !tag.completed)
      }
      return this.tags
    },
    unlockedAchievementsCount() {
      return this.achievements.filter(a => a.completed).length
    }
  },
  methods: {
    initMusic() {
      try {
        const music = document.getElementById('backgroundMusic')
        if (music) {
          music.volume = 0.3 // 设置音量为30%
          
          // 尝试自动播放
          music.play().then(() => {
            this.isMusicPlaying = true
            console.log('音乐自动播放成功')
          }).catch(error => {
            console.error('音乐自动播放失败:', error)
            this.isMusicPlaying = false
            
            // 当自动播放失败时，添加用户交互事件监听器
            // 一旦用户与页面交互，就尝试播放音乐
            const handleUserInteraction = () => {
              music.play().then(() => {
                this.isMusicPlaying = true
                console.log('音乐在用户交互后播放成功')
              }).catch(err => {
                console.error('用户交互后音乐播放仍然失败:', err)
              })
              
              // 移除事件监听器，避免重复触发
              document.removeEventListener('click', handleUserInteraction)
              document.removeEventListener('touchstart', handleUserInteraction)
              document.removeEventListener('keydown', handleUserInteraction)
            }
            
            // 添加多种用户交互事件监听器
            document.addEventListener('click', handleUserInteraction)
            document.addEventListener('touchstart', handleUserInteraction)
            document.addEventListener('keydown', handleUserInteraction)
          })
        }
      } catch (error) {
        console.error('初始化音乐失败:', error)
        this.isMusicPlaying = false
      }
    },
    toggleMusic() {
      try {
        const music = document.getElementById('backgroundMusic')
        if (music) {
          if (this.isMusicPlaying) {
            music.pause()
          } else {
            music.play().catch(error => {
              console.error('音乐播放失败:', error)
            })
          }
          this.isMusicPlaying = !this.isMusicPlaying
        }
      } catch (error) {
        console.error('切换音乐状态失败:', error)
      }
    },
    startOpeningAnimation() {
      let delay = 0
      this.openingLines.forEach((_, index) => {
        setTimeout(() => {
          this.animatedLines.push(index)
          // 播放打字机音效
          this.playTypewriterSound()
        }, delay)
        delay += index === 0 ? 1500 : 1200 // 放慢动画节奏，让文字有足够时间显示
      })
      
      // 所有文字显示完成后，延迟2秒再隐藏开场层
      setTimeout(() => {
        this.showOpening = false
      }, delay + 2000)
    },
    playTypewriterSound() {
      const audio = document.getElementById('typewriterSound')
      if (audio) {
        audio.currentTime = 0
        audio.play().catch(e => console.log('音效播放失败:', e))
      }
    },
    loadFromShareLink() {
      const urlParams = new URLSearchParams(window.location.search)
      const shareData = urlParams.get('share')
      if (shareData) {
        try {
          const decodedData = JSON.parse(atob(shareData))
          if (decodedData.tags) {
            this.tags = decodedData.tags
            this.saveToLocalStorage()
          }
          if (decodedData.theme) {
            this.theme = decodedData.theme
            this.saveTheme()
          }
        } catch (error) {
          console.error('解析分享数据失败:', error)
        }
      }
    },
    loadTheme() {
      const savedTheme = localStorage.getItem('lifeBucketListTheme')
      if (savedTheme) {
        this.theme = savedTheme
      }
      if (this.theme === 'dark') {
        document.body.classList.add('dark')
      } else {
        document.body.classList.remove('dark')
      }
    },
    saveTheme() {
      localStorage.setItem('lifeBucketListTheme', this.theme)
      if (this.theme === 'dark') {
        document.body.classList.add('dark')
      } else {
        document.body.classList.remove('dark')
      }
    },
    changeTheme(newTheme) {
      this.theme = newTheme
      this.saveTheme()
    },
    loadFromLocalStorage() {
      const savedTags = localStorage.getItem('lifeBucketList')
      if (savedTags) {
        this.tags = JSON.parse(savedTags)
      }
    },
    saveToLocalStorage() {
      localStorage.setItem('lifeBucketList', JSON.stringify(this.tags))
    },
    toggleTag(id) {
      try {
        // 使用id找到标签的索引
        const index = this.tags.findIndex(tag => tag.id === id)
        if (index !== -1) {
          // 使用Vue的响应式更新方法
          const newTags = [...this.tags]
          newTags[index] = {
            ...newTags[index],
            completed: !newTags[index].completed
          }
          this.tags = newTags
          this.saveToLocalStorage()
          this.checkAchievements()
        }
      } catch (error) {
        console.error('切换标签状态失败:', error)
      }
    },
    resetTags() {
      try {
        // 重置所有标签的完成状态
        const newTags = this.tags.map(tag => ({
          ...tag,
          completed: false
        }))
        this.tags = newTags
        
        // 重置所有成就
        const newAchievements = this.achievements.map(achievement => ({
          ...achievement,
          completed: false
        }))
        this.achievements = newAchievements
        
        // 保存数据到本地存储
        this.saveToLocalStorage()
        this.saveAchievements()
        
        // 显示安慰提示
        this.show安慰 = true
        setTimeout(() => {
          this.show安慰 = false
        }, 3000)
      } catch (error) {
        console.error('重置标签失败:', error)
      }
    },
    addTag() {
      if (this.newTagText.trim()) {
        // 生成唯一的id
        const maxId = this.tags.length > 0 ? Math.max(...this.tags.map(tag => tag.id)) : 0
        const newId = maxId + 1
        this.tags.push({ id: newId, text: this.newTagText.trim(), completed: false })
        this.newTagText = ''
        this.saveToLocalStorage()
      }
    },
    deleteTag(id) {
      try {
        // 使用id找到标签的索引
        const index = this.tags.findIndex(tag => tag.id === id)
        if (index !== -1) {
          // 使用Vue的响应式更新方法
          const newTags = [...this.tags]
          newTags.splice(index, 1)
          this.tags = newTags
          this.saveToLocalStorage()
        }
      } catch (error) {
        console.error('删除标签失败:', error)
      }
    },
    startEdit(id) {
      this.editingId = id
      // 使用id找到标签的索引
      const index = this.tags.findIndex(tag => tag.id === id)
      if (index !== -1) {
        this.editText = this.tags[index].text
        this.$nextTick(() => {
          const inputRef = this.$refs[`editInput_${id}`]
          if (inputRef && inputRef[0]) {
            inputRef[0].focus()
          }
        })
      }
    },
    finishEdit() {
      try {
        if (this.editingId !== -1 && this.editText.trim()) {
          // 使用id找到标签的索引
          const index = this.tags.findIndex(tag => tag.id === this.editingId)
          if (index !== -1) {
            // 使用Vue的响应式更新方法
            const newTags = [...this.tags]
            newTags[index] = {
              ...newTags[index],
              text: this.editText.trim()
            }
            this.tags = newTags
            this.saveToLocalStorage()
            this.editingId = -1
          }
        }
      } catch (error) {
        console.error('完成编辑标签失败:', error)
      }
    },
    cancelEdit() {
      this.editingId = -1
    },
    checkAchievements() {
      let hasNewAchievement = false
      this.achievements.forEach((achievement, index) => {
        if (!achievement.completed) {
          let unlocked = false
          if (achievement.requirement.type === 'count') {
            unlocked = this.completedCount >= achievement.requirement.value
          } else if (achievement.requirement.type === 'tags') {
            unlocked = achievement.requirement.values.every(tagText => {
              const tag = this.tags.find(t => t.text === tagText)
              return tag && tag.completed
            })
          } else if (achievement.requirement.type === 'category') {
            const categoryTags = this.getCategoryTags(achievement.requirement.value)
            const completedCategoryTags = categoryTags.filter(tag => tag.completed)
            unlocked = completedCategoryTags.length >= achievement.requirement.count
          }
          if (unlocked) {
            this.achievements[index].completed = true
            this.currentAchievement = achievement
            this.showAchievementPopup = true
            hasNewAchievement = true
            setTimeout(() => {
              this.showAchievementPopup = false
            }, 3000)
          }
        }
      })
      if (hasNewAchievement) {
        this.saveAchievements()
      }
    },
    getCategoryTags(category) {
      const categories = {
        social: ['送礼物', '被送礼物', '暗恋', '明恋', '失恋', '表白', '被表白', '拒绝他人表白', '被表白', '当海王', '拥有要好的异性朋友', '谈恋爱', '和朋友旅游', '社团活动', '和同学打水仗'],
        adventure: ['跳伞', '坐过山车', '蹦极', '骑马', '卡丁车', '攀岩', '滑雪', '登山', '野炊', '一个人旅游']
      }
      const categoryTexts = categories[category] || []
      return this.tags.filter(tag => categoryTexts.includes(tag.text))
    },
    saveAchievements() {
      localStorage.setItem('lifeBucketListAchievements', JSON.stringify(this.achievements))
    },
    loadAchievements() {
      const savedAchievements = localStorage.getItem('lifeBucketListAchievements')
      if (savedAchievements) {
        this.achievements = JSON.parse(savedAchievements)
      }
    },
    toggleAchievementPanel() {
      this.showAchievementPanel = !this.showAchievementPanel
    },
    randomChallenge() {
      const uncompletedTags = this.tags.filter(tag => !tag.completed)
      if (uncompletedTags.length > 0) {
        const randomIndex = Math.floor(Math.random() * uncompletedTags.length)
        // 深拷贝挑战标签，避免引用问题
        this.currentChallenge = JSON.parse(JSON.stringify(uncompletedTags[randomIndex]))
        setTimeout(() => {
          this.currentChallenge = null
        }, 5000)
      }
    },
    async shareList() {
      try {
        // 临时打开成就面板，确保截图时面板是打开的
        const wasAchievementPanelOpen = this.showAchievementPanel
        this.showAchievementPanel = true
        
        // 等待DOM更新和成就面板完全展开
        await this.$nextTick()
        // 增加额外等待时间，确保成就面板完全渲染
        await new Promise(resolve => setTimeout(resolve, 500))
        
        const container = document.querySelector('.container')
        if (container) {
          // 获取容器的实际高度，包括成就面板
          const actualHeight = container.scrollHeight
          // 临时设置容器高度，确保成就面板不被截断
          container.style.minHeight = actualHeight + 'px'
          
          const canvas = await html2canvas(container, {
            scale: 2,
            useCORS: true,
            backgroundColor: this.theme === 'dark' ? '#1a1a1a' : '#f5f7fa',
            // 增加滚动捕获，确保所有内容都被捕获
            scrolling: 'no',
            // 确保成就面板在截图范围内
            windowWidth: window.innerWidth,
            windowHeight: window.innerHeight
          })
          const imageUrl = canvas.toDataURL('image/png')
          const link = document.createElement('a')
          link.href = imageUrl
          link.download = '人生清单分享.png'
          link.click()
          
          // 恢复容器高度设置
          container.style.minHeight = ''
        }
        
        // 恢复成就面板状态
        this.showAchievementPanel = wasAchievementPanelOpen
      } catch (error) {
        console.error('生成分享图片失败:', error)
      }
    },
    generateShareLink() {
      const shareData = {
        tags: this.tags,
        theme: this.theme
      }
      const shareString = btoa(JSON.stringify(shareData))
      const currentUrl = window.location.origin + window.location.pathname
      const shareUrl = `${currentUrl}?share=${shareString}`
      navigator.clipboard.writeText(shareUrl).then(() => {
        alert('分享链接已复制到剪贴板！')
      }).catch(err => {
        console.error('复制失败:', err)
        alert('分享链接生成失败，请手动复制：' + shareUrl)
      })
    }
  }
}
</script>

<style>
/* 样式已在style.css中定义 */
</style>