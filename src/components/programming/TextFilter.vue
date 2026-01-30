<template>
  <div>
    <a-card>
      <h1>文本过滤工具</h1>

      <div class="controls">
        <div class="filter-keywords">
          <label>过滤关键词 (用逗号分隔):</label>
          <input 
            type="text" 
            v-model="filterKeywords" 
            placeholder="del, remove, delete, drop"
            class="keywords-input"
          />
        </div>
        <label class="wrap-label">
          <input type="checkbox" v-model="lineWrap" @change="toggleLineWrap" />
          自动换行
        </label>
        <label class="case-sensitive-label">
          <input type="checkbox" v-model="caseSensitive" />
          区分大小写
        </label>
        <label class="trim-label">
          <input type="checkbox" v-model="trimWhitespace" />
          去除首尾空格
        </label>
        <button @click="filterText">过滤处理</button>
        <button @click="clearAll">清空</button>
      </div>

      <div class="editor-container">
        <div class="editor-box">
          <label>原始文本</label>
          <textarea ref="inputEditor"></textarea>
        </div>
        <div class="editor-box">
          <label>过滤结果</label>
          <textarea ref="outputEditor" readonly></textarea>
        </div>
      </div>

      <div class="stats">
        <div class="stat-item">
          <span class="stat-label">原始行数:</span>
          <span class="stat-value">{{ originalLineCount }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">过滤后行数:</span>
          <span class="stat-value">{{ filteredLineCount }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">删除行数:</span>
          <span class="stat-value">{{ deletedLineCount }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">过滤率:</span>
          <span class="stat-value">{{ filterRate }}%</span>
        </div>
      </div>
    </a-card>
  </div>
</template>

<script>
import { onMounted, ref, computed } from "vue";
import CodeMirror from "codemirror";
import "codemirror/lib/codemirror.css";
import "codemirror/mode/javascript/javascript";

export default {
  setup() {
    const inputEditor = ref(null);
    const outputEditor = ref(null);
    const lineWrap = ref(true);
    const caseSensitive = ref(false);
    const trimWhitespace = ref(true);
    const filterKeywords = ref("del, remove");
    const originalLineCount = ref(0);
    const filteredLineCount = ref(0);

    let inputCodeMirror = null;
    let outputCodeMirror = null;

    const deletedLineCount = computed(() => {
      return originalLineCount.value - filteredLineCount.value;
    });

    const filterRate = computed(() => {
      if (originalLineCount.value === 0) return 0;
      return Math.round((deletedLineCount.value / originalLineCount.value) * 100);
    });

    onMounted(() => {
      inputCodeMirror = CodeMirror.fromTextArea(inputEditor.value, {
        lineNumbers: true,
        mode: "javascript",
        theme: "default",
        lineWrapping: true,
        viewportMargin: Infinity,
        placeholder: "请输入需要过滤的文本，每行一个条目..."
      });

      outputCodeMirror = CodeMirror.fromTextArea(outputEditor.value, {
        lineNumbers: true,
        mode: "javascript",
        theme: "default",
        lineWrapping: true,
        viewportMargin: Infinity,
        readOnly: true,
        placeholder: "过滤结果将显示在这里..."
      });

      // 监听输入变化
      inputCodeMirror.on('change', () => {
        updateStats();
      });
    });

    const toggleLineWrap = () => {
      inputCodeMirror.setOption("lineWrapping", lineWrap.value);
      outputCodeMirror.setOption("lineWrapping", lineWrap.value);
    };

    const updateStats = () => {
      const text = inputCodeMirror.getValue();
      const lines = text.split('\n').filter(line => line.trim() !== '');
      originalLineCount.value = lines.length;
    };

    const filterText = () => {
      const inputText = inputCodeMirror.getValue();
      const lines = inputText.split('\n');
      
      // 解析过滤关键词
      const keywords = filterKeywords.value
        .split(',')
        .map(keyword => keyword.trim())
        .filter(keyword => keyword.length > 0);

      if (keywords.length === 0) {
        alert('请输入至少一个过滤关键词');
        return;
      }

      // 处理每一行
      const processedLines = lines.map(line => {
        if (trimWhitespace.value) {
          return line.trim();
        }
        return line;
      });

      // 过滤逻辑
      const filteredLines = processedLines.filter(line => {
        const searchText = caseSensitive.value ? line : line.toLowerCase();
        
        // 检查是否包含任何关键词
        return !keywords.some(keyword => {
          const searchKeyword = caseSensitive.value ? keyword : keyword.toLowerCase();
          return searchText.includes(searchKeyword);
        });
      });

      // 更新输出编辑器
      const result = filteredLines.join('\n');
      outputCodeMirror.setValue(result);
      
      // 更新统计信息
      filteredLineCount.value = filteredLines.length;
    };

    const clearAll = () => {
      inputCodeMirror.setValue('');
      outputCodeMirror.setValue('');
      originalLineCount.value = 0;
      filteredLineCount.value = 0;
    };

    return {
      inputEditor,
      outputEditor,
      lineWrap,
      caseSensitive,
      trimWhitespace,
      filterKeywords,
      originalLineCount,
      filteredLineCount,
      deletedLineCount,
      filterRate,
      toggleLineWrap,
      filterText,
      clearAll
    };
  },
};
</script>

<style scoped>
body {
  font-family: Arial, sans-serif;
  margin: 20px;
  background: #f9f9f9;
}

h1 {
  text-align: center;
  margin-bottom: 15px;
}

.editor-container {
  display: flex;
  margin: 0 auto;
  height: 600px;
  border: 1px solid #ddd;
  background: white;
}

.editor-box {
  width: 50%;
  display: flex;
  flex-direction: column;
}

.editor-box label {
  padding: 8px 12px;
  background: #f0f0f0;
  border-bottom: 1px solid #ddd;
  font-weight: bold;
}

:deep(.CodeMirror) {
  flex-grow: 1;
  height: 100% !important;
  font-size: 14px;
}

.controls {
  max-width: 1200px;
  margin: 10px auto 30px;
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  flex-wrap: wrap;
}

.filter-keywords {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  min-width: 300px;
}

.filter-keywords label {
  font-weight: bold;
  color: #333;
  font-size: 14px;
}

.keywords-input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background: white;
}

.keywords-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

button {
  font-size: 16px;
  padding: 8px 18px;
  cursor: pointer;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 4px;
}

button:hover {
  background-color: #0056b3;
}

button:last-of-type {
  background-color: #6c757d;
}

button:last-of-type:hover {
  background-color: #545b62;
}

.wrap-label,
.case-sensitive-label,
.trim-label {
  font-weight: normal;
  user-select: none;
  display: flex;
  align-items: center;
  gap: 6px;
}

.wrap-label input,
.case-sensitive-label input,
.trim-label input {
  margin: 0;
}

.stats {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-top: 20px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.stat-label {
  font-size: 14px;
  color: #666;
  font-weight: 500;
}

.stat-value {
  font-size: 18px;
  font-weight: bold;
  color: #007bff;
}

@media (max-width: 768px) {
  .editor-container {
    flex-direction: column;
    height: auto;
  }
  
  .editor-box {
    width: 100%;
    height: 300px;
  }
  
  .controls {
    flex-direction: column;
    gap: 10px;
  }
  
  .filter-keywords {
    min-width: auto;
    width: 100%;
  }
  
  .stats {
    flex-direction: column;
    gap: 15px;
  }
}
</style>
