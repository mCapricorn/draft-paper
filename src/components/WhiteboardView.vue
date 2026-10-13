<template>
  <div class="whiteboard-root">
    <!-- Debug toggle button (fixed top-left) -->
    <button @click="toggleDebugPanel"
            title="调试设置"
            style="position:fixed; left:8px; top:8px; z-index:1000; padding:6px 10px; border-radius:8px; border:1px solid rgba(0,0,0,0.14); background:rgba(255,255,255,0.75); backdrop-filter: blur(6px); box-shadow:0 2px 10px rgba(0,0,0,0.10); cursor:pointer; color:#111827;">
      Debug
    </button>
    <!-- Debug panel -->
    <div v-if="debugOpen" style="position:fixed; left:8px; top:44px; z-index:1000; width:320px; max-height:80vh; overflow:auto; padding:12px; border-radius:12px; border:1px solid rgba(0,0,0,0.16); background:rgba(255,255,255,0.72); backdrop-filter: blur(8px); box-shadow:0 8px 24px rgba(0,0,0,0.14); color:#111827;">
      <div style="font-weight:700; margin-bottom:8px;">调试设置</div>
      <div class="dbg-row" style="gap:8px; margin-bottom:10px; display:flex;">
        <button @click="saveDebugPreset" style="flex:1; padding:6px 8px; border-radius:8px; border:1px solid rgba(0,0,0,0.14); background:rgba(16,185,129,0.12); color:#065f46; cursor:pointer;">保存预设</button>
        <button @click="loadDebugPreset" style="flex:1; padding:6px 8px; border-radius:8px; border:1px solid rgba(0,0,0,0.14); background:rgba(59,130,246,0.12); color:#1e3a8a; cursor:pointer;">恢复预设</button>
      </div>
      <div style="font-size:12px; color:#374151; margin-bottom:6px; opacity:0.8;">形状识别</div>
      <div class="dbg-row">
        <label>简化比例 (diag×)</label>
        <input type="range" min="0.008" max="0.040" step="0.001" v-model.number="dbg.simplifyScale"/>
        <span class="dbg-val">{{dbg.simplifyScale.toFixed(3)}}</span>
      </div>
      <div class="dbg-row">
        <label>简化像素下限</label>
        <input type="range" min="0" max="10" step="1" v-model.number="dbg.minSimplifyPx"/>
        <span class="dbg-val">{{dbg.minSimplifyPx}}</span>
      </div>
      <div class="dbg-row">
        <label>简化像素上限</label>
        <input type="range" min="2" max="20" step="1" v-model.number="dbg.maxSimplifyPx"/>
        <span class="dbg-val">{{dbg.maxSimplifyPx}}</span>
      </div>
      <div class="dbg-row">
        <label>角点最小段比</label>
        <input type="range" min="0.005" max="0.060" step="0.001" v-model.number="dbg.cornerMinSegRatio"/>
        <span class="dbg-val">{{(dbg.cornerMinSegRatio*100).toFixed(1)}}%</span>
      </div>
      <div class="dbg-row">
        <label>角点阈值 (°)</label>
        <input type="range" min="10" max="60" step="1" v-model.number="dbg.cornerAngleDeg"/>
        <span class="dbg-val">{{dbg.cornerAngleDeg}}°</span>
      </div>
      <div class="dbg-row">
        <label>直线化剔除 (°)</label>
        <input type="range" min="3" max="20" step="1" v-model.number="dbg.pruneAngleDeg"/>
        <span class="dbg-val">{{dbg.pruneAngleDeg}}°</span>
      </div>
      <div class="dbg-row">
        <label>闭合阈值 (diag×)</label>
        <input type="range" min="0.03" max="0.12" step="0.005" v-model.number="dbg.closedThreshRatio"/>
        <span class="dbg-val">{{(dbg.closedThreshRatio*100).toFixed(1)}}%</span>
      </div>
      <div class="dbg-row">
        <label>直线偏差阈值 (diag×)</label>
        <input type="range" min="0.005" max="0.040" step="0.001" v-model.number="dbg.lineDeviationRatio"/>
        <span class="dbg-val">{{(dbg.lineDeviationRatio*100).toFixed(1)}}%</span>
      </div>
      <div style="font-size:12px; color:#374151; margin:10px 0 6px; opacity:0.8;">接缝软化</div>
      <div class="dbg-row">
        <label>模糊半径 (px)</label>
        <input type="range" min="0" max="2" step="0.05" v-model.number="dbg.seamBlurPx"/>
        <span class="dbg-val">{{dbg.seamBlurPx.toFixed(2)}}px</span>
      </div>
      <div class="dbg-row">
        <label>模糊透明度</label>
        <input type="range" min="0" max="1" step="0.05" v-model.number="dbg.seamBlurAlpha"/>
        <span class="dbg-val">{{dbg.seamBlurAlpha.toFixed(2)}}</span>
      </div>
      <div class="dbg-row">
        <label>接缝点半径×笔粗</label>
        <input type="range" min="0.2" max="1.2" step="0.05" v-model.number="dbg.seamDotScale"/>
        <span class="dbg-val">×{{dbg.seamDotScale.toFixed(2)}}</span>
      </div>
    </div>
    <aside class="wb-left" :style="leftStyle">
      <button class="wb-btn" :class="{active: panMode}" @click="togglePanMode" @mouseenter="onPanHover(true)" @mouseleave="onPanHover(false)" title="手型：拖动画布（按住滚轮也可拖动）">
        <!-- open hand icon -->
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M7 11V6a2 2 0 0 1 4 0v5"/>
          <path d="M11 11V5a2 2 0 0 1 4 0v6"/>
          <path d="M15 11V7a2 2 0 0 1 4 0v7a5 5 0 0 1-5 5h-2a5 5 0 0 1-5-5v-3"/>
        </svg>
      </button>
      <button class="wb-btn" @click="zoomIn" title="放大">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="11" cy="11" r="6"/>
          <path d="M11 8v6"/>
          <path d="M8 11h6"/>
          <path d="M21 21l-4.35-4.35"/>
        </svg>
      </button>
      <button class="wb-btn" @click="zoomOut" title="缩小">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="11" cy="11" r="6"/>
          <path d="M8 11h6"/>
          <path d="M21 21l-4.35-4.35"/>
        </svg>
      </button>
      <button class="wb-btn" @click="fitToView" title="看全（使整幅画布完全显示）">
        <!-- fit-to-view icon: frame with corners -->
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M3 9V5a2 2 0 0 1 2-2h4"/>
          <path d="M21 9V5a2 2 0 0 0-2-2h-4"/>
          <path d="M3 15v4a2 2 0 0 0 2 2h4"/>
          <path d="M21 15v4a2 2 0 0 1-2 2h-4"/>
          <rect x="7.5" y="7.5" width="9" height="9" rx="1"/>
        </svg>
      </button>
      <button class="wb-btn" :class="{active: shapeSnapEnabled}" @click="toggleShapeSnap" :title="shapeSnapEnabled ? '形状优化：开' : '形状优化：关'">
        <!-- magic-wand icon -->
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M15 9l-6 6"/>
          <path d="M11 7l2-2"/>
          <path d="M17 13l2-2"/>
          <path d="M8 3l1 1"/>
          <path d="M3 8l1 1"/>
          <path d="M19 5l1 1"/>
          <path d="M4 20l6-6"/>
        </svg>
      </button>
      <div class="rotate-vertical" title="旋转">
        <!-- 上：顺时针 90° -->
        <button class="wb-btn rotate-cw" @click.prevent="rotate90('cw')" title="顺时针 90°">
          <!-- clearer rotate-90-cw icon -->
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M7 11V7a4 4 0 0 1 4-4h4"/>
            <path d="M15 3l3 3-3 3"/>
            <path d="M7 13a7 7 0 1 0 7-7"/>
          </svg>
        </button>
        <!-- 下：慢速旋转（按住并滚轮） -->
        <div class="slow-rotate-wrap">
            <button class="wb-btn rotate-slow" @mousedown.prevent="onRotateMouseDown('cw', $event)" @mouseup.prevent="onRotateMouseUp" @mouseenter="onSlowRotateHover(true)" @mouseleave="onSlowRotateHover(false)" title="慢速旋转（按住并滑动滚轮）">
            <!-- clearer slow-rotate icon with circular arrow -->
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M21 12a9 9 0 1 1-9-9"/>
              <path d="M15 3l3 3-3 3"/>
            </svg>
          </button>
            <div class="rotate-hover-hint right" :class="{show: showRotateHoverHint}" :style="{ color: hintTextColor }">或在画布内按住右键滑动滚轮（以指针为圆心）</div>
        </div>
      </div>

      <div class="wb-sep"></div>

      <div class="texture-wrap" title="画布材质">
        <div class="texture-caption" :style="{ color: hintTextColor }">材质</div>
        <select class="texture-select" v-model="texture" @change="onTextureChange">
          <option value="none">无</option>
          <option value="grid">网格</option>
          <option value="paper">纸张</option>
        </select>
      </div>

      <label class="wb-color">
        <!-- 画布颜色改为圆形选择按钮 -->
        <input class="round-bg-color" type="color" v-model="bgColor" @input="onBgColorChange" title="画布颜色" />
      </label>
        <div class="wb-color-palette">
          <!-- 护眼色下拉，与材质选择一致样式 -->
          <div class="eye-select-wrap">
            <!-- 护眼色使用与材质相同的选择框样式并加宽 -->
            <select class="texture-select eye-select" v-model="_selectedEyeColor" @change="onEyeSelectChange" :title="'选择护眼色'">
              <option value="">关|护眼</option>
              <option v-for="(eCol, i) in eyeProtectColors" :key="'eoptL'+i" :value="eCol.hex">{{ eCol.name }}</option>
            </select>
          </div>
        </div>
    </aside>

    <section class="wb-canvas-area">
      <div class="wb-canvas-wrap" :style="canvasWrapStyle" ref="wrap">
  <canvas ref="canvas" class="wb-canvas" :width="viewW" :height="viewH"></canvas>
        <!-- top-right display fullscreen button -->
        <button class="wb-exit-page-btn" @click="exitWhiteboard" title="退出稿纸，返回上页">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M15 3h4a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-4"/>
            <path d="M10 17l-5-5 5-5"/>
            <path d="M5 12h12"/>
          </svg>
        </button>
        <button class="wb-fullscreen-btn" @click="toggleDisplayFullscreen" :title="isDisplayFullscreen ? '退出全屏 (Esc)' : '在显示器全屏'">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 9V3h6"/><path d="M21 15v6h-6"/></svg>
        </button>
        <!-- top-left exit when in fullscreen -->
        <button v-if="isDisplayFullscreen" class="wb-exit-btn" @click="toggleDisplayFullscreen" title="退出全屏">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 3H3v6"/><path d="M15 21h6v-6"/><path d="M3 3l7 7"/><path d="M21 21l-7-7"/></svg>
        </button>
      </div>
    </section>

    <!-- HUD -->
    <div v-if="_hudVisible" class="wb-hud show">{{_hudMessage}}</div>

  <aside class="wb-right" :class="{ 'wb-right-wide': (pasteActive && pasteStroke && pasteStroke.type==='text') }" :style="rightStyle">
  <button class="wb-btn" @click="undo" title="撤销 (Ctrl+Z)"><svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17L4 12l5-5"/><path d="M20 18a8 8 0 0 0-8-8H4"/></svg></button>
  <button class="wb-btn" @click="redo" title="重做 (Ctrl+Shift+Z)"><svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M15 17l5-5-5-5"/><path d="M4 18a8 8 0 0 1 8-8h8"/></svg></button>
      <button class="wb-btn" @click="clearCanvas" title="清空"><svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 6h18"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6"/><path d="M14 11v6"/></svg></button>

      <div class="wb-sep"></div>

      <button class="wb-btn" @click="screenshot" title="截图">截图</button>
      <label class="wb-btn" title="导入">
        <input ref="importFileInput" class="hidden-file" type="file" accept=".zxkjbb,.json" @change="onImportFile"> 导入
      </label>
      <button class="wb-btn" @click="exportBoard" title="导出">导出</button>

      <div class="wb-sep"></div>

      <label class="wb-btn" title="制作数字拼图" v-if="false">
        <input ref="puzzleFile" class="hidden-file" type="file" accept="image/*" @change="onPuzzleFileChange"> 数字拼图
      </label>

      <!-- 文本样式面板：仅在粘贴文本预览时显示 -->
      <div v-if="pasteActive && pasteStroke && pasteStroke.type==='text'" class="text-style-panel">
        <div class="panel-title">文本样式</div>
        <div class="row">
          <label>字体</label>
          <select v-model="pasteStroke.fontFamily" @change="onTextStyleChange">
            <option value="sans-serif">无衬线（Sans-Serif）</option>
            <option value="serif">衬线（Serif）</option>
            <option value="monospace">等宽（Monospace）</option>
            <option value="'Segoe UI', Arial, sans-serif">Segoe UI</option>
            <option value="Arial, Helvetica, sans-serif">Arial</option>
            <option value="'Microsoft YaHei', 'PingFang SC', 'Hiragino Sans GB', 'Noto Sans SC', 'Heiti SC', sans-serif">中文黑体/苹方/雅黑</option>
            <option value="'Times New Roman', Times, serif">Times New Roman</option>
            <option value="Georgia, serif">Georgia</option>
            <option value="'Courier New', Courier, monospace">Courier New</option>
            <option value="Consolas, Menlo, monospace">Consolas</option>
            <option value="Roboto, Helvetica, Arial, sans-serif">Roboto</option>
          </select>
        </div>
        <div class="row">
          <label>粗细</label>
          <select v-model="pasteStroke.fontWeight" @change="onTextStyleChange">
            <option value="normal">正常</option>
            <option value="bold">加粗</option>
          </select>
        </div>
        <div class="row">
          <label>字号</label>
          <input type="range" min="8" max="160" step="1" v-model.number="pasteStroke.baseSize" @input="onTextStyleChange" />
          <span class="val">{{ pasteStroke.baseSize }}px</span>
        </div>
        <div class="row">
          <label>对齐</label>
          <div class="align-group">
            <button type="button" :class="{active: pasteStroke.align==='left'}" @click="pasteStroke.align='left'; onTextStyleChange()">左</button>
            <button type="button" :class="{active: pasteStroke.align==='center'}" @click="pasteStroke.align='center'; onTextStyleChange()">中</button>
            <button type="button" :class="{active: pasteStroke.align==='right'}" @click="pasteStroke.align='right'; onTextStyleChange()">右</button>
          </div>
        </div>
        <div class="row">
          <label>填满框</label>
          <input type="checkbox" v-model="pasteStroke.fillBox" @change="onTextStyleChange" />
        </div>
        <div class="row">
          <label>文字</label>
          <input type="color" v-model="pasteStroke.color" @input="onTextStyleChange" />
        </div>
        <div class="row">
          <label>背景</label>
          <input type="color" v-model="pasteStroke.bgColor" @input="onTextStyleChange" />
        </div>
        <div class="row">
          <label>描边色</label>
          <input type="color" v-model="pasteStroke.textStrokeColor" @input="onTextStyleChange" />
        </div>
        <div class="row">
          <label>描边宽</label>
          <input type="range" min="0" max="6" step="0.5" v-model.number="pasteStroke.textStrokeWidth" @input="onTextStyleChange" />
          <span class="val">{{ (pasteStroke.textStrokeWidth||0).toFixed(1) }}px</span>
        </div>
        <div class="row">
          <label>行距</label>
          <input type="range" min="0.8" max="2.4" step="0.05" v-model.number="pasteStroke.lineHeightMult" @input="onTextStyleChange" />
          <span class="val">×{{ (pasteStroke.lineHeightMult||1.2).toFixed(2) }}</span>
        </div>
        <div class="row">
          <label>字距</label>
          <input type="range" min="-2" max="20" step="0.5" v-model.number="pasteStroke.letterSpacing" @input="onTextStyleChange" />
          <span class="val">{{ (pasteStroke.letterSpacing||0).toFixed(1) }}px</span>
        </div>
        <div class="row">
          <label>阴影</label>
          <input type="color" v-model="pasteStroke.textShadowColor" @input="onTextStyleChange" />
        </div>
        <div class="row">
          <label>模糊</label>
          <input type="range" min="0" max="20" step="0.5" v-model.number="pasteStroke.textShadowBlur" @input="onTextStyleChange" />
          <span class="val">{{ (pasteStroke.textShadowBlur||0).toFixed(1) }}px</span>
        </div>
        <div class="row">
          <label>X偏移</label>
          <input type="range" min="-40" max="40" step="1" v-model.number="pasteStroke.textShadowOffsetX" @input="onTextStyleChange" />
          <span class="val">{{ (pasteStroke.textShadowOffsetX||0).toFixed(0) }}px</span>
        </div>
        <div class="row">
          <label>Y偏移</label>
          <input type="range" min="-40" max="40" step="1" v-model.number="pasteStroke.textShadowOffsetY" @input="onTextStyleChange" />
          <span class="val">{{ (pasteStroke.textShadowOffsetY||0).toFixed(0) }}px</span>
        </div>
      </div>
    </aside>

  <footer class="wb-bottom" :style="bottomStyle">
        <div class="brush-controls">
          <!-- 画笔区域：上方多个画笔按钮，下方画笔粗细 -->
          <div class="brush-group">
            <div class="brush-style">
              <button :class="['tool', {active: brushStyle==='pen'}]" @click="selectBrush('pen')" title="画笔">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20l7-7-5-5-7 7v5z"/><path d="M14 8l2 2"/></svg>
              </button>
              <button :class="['tool', {active: brushStyle==='marker'}]" @click="selectBrush('marker')" title="马克笔">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 21l9-9 4 4-9 9H3v-4z"/><path d="M14 7l3 3"/></svg>
              </button>
              <button :class="['tool', {active: brushStyle==='pencil'}]" @click="selectBrush('pencil')" title="铅笔">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20l9-9-3-3-9 9v3z"/><path d="M18 5l1 1"/></svg>
              </button>
            </div>
            <div class="brush-size">
                <input type="range" min="1" max="14" v-model.number="brushSize" title="按住CTRL+滑动滚轮调整大小" />
              <div class="size-ind" :style="{ color: hintTextColor }">{{brushSize}}</div>
            </div>
          </div>

          <!-- 画笔颜色（中间） -->
          <div class="color-group">
            <div class="brush-color">
              <!-- 点击/聚焦打开颜色选择器；选择期间不记录最近颜色，实际绘画后再记录 -->
			<div style="display:flex; align-items:center; gap:8px;">
				<input
					class="round-color"
					type="color"
					v-model="brushColor"
					@focus="onBrushPickerOpen"
					@blur="onBrushPickerClose"
					@input="onBrushColorChange"
					title="画笔颜色"
				/>
				<div class="bw-group" style="margin-left:8px;">
					<div style="display:flex; align-items:center; gap:8px;">
						<div
							class="swatch bw-swatch"
							@click="setBrushColor('#000000')"
							title="黑色"
							:style="{ background: '#000' }"
						></div>
						<div
							class="swatch bw-swatch"
							@click="setBrushColor('#ffffff')"
							title="白色"
							:style="{ background: '#fff', border: '1px solid rgba(0,0,0,0.08)' }"
						></div>
					</div>
				</div>
			</div></div>
            <!-- brush: black/white fixed on right as circular buttons; recent colors row with 5 slots (dashed placeholders if empty) -->
            <div class="brush-side-row">
              
              <div class="recent-row">
                <template v-for="i in 5">
                  <div v-if="recentBrushColors[i-1]" :key="'recent'+i" class="swatch recent-swatch" @click="setBrushColor(recentBrushColors[i-1])" :style="{ background: recentBrushColors[i-1] }"></div>
                  <div v-else :key="'ph'+i" class="swatch recent-placeholder"></div>
                </template>
              </div>
            </div>
          </div>
          <div class="eraser-group">
            <button :class="['tool', {active: erasing}]" @click="toggleEraser" :title="'按住空格以使用橡皮擦'">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 14l-6 6H6v-6l9-9 5 5z"/></svg>
            </button>
            <div class="brush-size eraser-size" style="margin-top:8px">
              <input type="range" min="2" max="32" v-model.number="eraserSize" title="按住空格+滑动滚轮调整大小" />
              <div class="size-ind" :style="{ color: hintTextColor }">{{eraserSize}}</div>
            </div>
          </div>
        </div>
    </footer>
  </div>
</template>

<script>
import JSZip from 'jszip'
export default {
  name: 'WhiteboardView',
  emits: ['request-exit', 'autosave-updated'],
  props: {
    boardTitle: {
      type: String,
      default: 'Shared Draft'
    },
    autosaveKey: {
      type: String,
      default: 'zx-whiteboard-autosave'
    },
    performanceMode: {
      type: String,
      default: 'balanced'
    }
  },
  watch: {
    performanceMode(mode) {
      const isHighPerformance = mode === 'high'
      this.maxDisplayScale = isHighPerformance ? 3 : 4
      this.dbg = {
        ...this.dbg,
        minSimplifyPx: isHighPerformance ? 4 : 2,
        maxSimplifyPx: isHighPerformance ? 10 : 8,
        simplifyScale: isHighPerformance ? 0.018 : 0.016,
        seamBlurPx: isHighPerformance ? 0.4 : 0.6,
        seamDotScale: isHighPerformance ? 0.45 : 0.55
      }
      this.invalidate && this.invalidate()
    }
  },
  data() {
    return {
      // default to viewport size so canvas fills browser
      width: 1600,
      height: 900,
      viewW: (typeof window !== 'undefined') ? window.innerWidth : 1600,
      viewH: (typeof window !== 'undefined') ? window.innerHeight : 900,
  // if true, keep logical board size matched to viewport (so you can draw everywhere)
  _autoFitContent: true,
    // when auto-fit is enabled, make the logical board N times of viewport
    _autoFitLogicalMultiplier: 4,
    // 清空复位后的初始视角放大系数（相对 contain），>1 表示更近一些
    _resetCloserFactor: 10,
    // device pixel ratio for high-DPI rendering
    _dpr: (typeof window !== 'undefined') ? (window.devicePixelRatio || 1) : 1,
    // min zoom scale to prevent infinite zoom-out
    minScale: 0.25,
  // desired display scale bounds (independent于逻辑画布大小变化)
  maxDisplayScale: 4,
  minDisplayScale: 0.25,
    // expansion buffer (logical pixels) for infinite-canvas growth
    _expandBuffer: 1024,
    // maximum backing pixels per dimension for offscreen (safety across browsers)
    _maxBackingPixels: 16384,
    // computed caps for logical content size based on DPR (set in resize)
    _maxLogicalW: 8192,
    _maxLogicalH: 8192,
  // user zoom factor (combined with auto-fit). With 4x logical board, 1x here shows entire board at 0.25x initially
  scale: 1,
      rotation: 0, // degrees
      texture: 'none',
      bgColor: '#ffffff',
      brushColor: '#000000',
  brushSize: 2,
      eraserSize: 24,
      brushStyle: 'pen',
      erasing: false,
  // shape snapping (beautification)
  shapeSnapEnabled: true,
  _shapePreview: null,
  _shapePreviewTimer: null,
  _previewAnchor: null,
  _previewMoved: false,
  // hold-to-preview timers while still pressing after drawing motion stops
  _shapeHoldPreviewTimer: null,
  _shapeHoldCommitTimer: null,
  _shapeHoldAnchor: null,
  _suppressNextPointerUp: false,
  // redraw coalescing
  _rafPending: false,
      // Debug UI
      debugOpen: false,
      dbg: {
        // simplify tolerance = clamp(min,max, diag * simplifyScale)
        simplifyScale: 0.016,
        minSimplifyPx: 2,
        maxSimplifyPx: 8,
        // corners
        cornerMinSegRatio: 0.02,
        cornerAngleDeg: 33,
        pruneAngleDeg: 10,
        // recognition thresholds
        closedThreshRatio: 0.06,
        lineDeviationRatio: 0.015,
        // seam softening
        seamBlurPx: 0.6,
        seamBlurAlpha: 0.5,
        seamDotScale: 0.55
      },
      // pan (hand) tool state
      panMode: false,
      isPanning: false,
      panX: 0,
      panY: 0,
      _lastPanWorld: null,
      _lastPanScreen: null,
      _prePanMode: false,
      // Space 临时橡皮擦
      _spaceEraserActive: false,
      _preErasing: false,
      _preBrushStyle: 'pen',
      // 分层绘制：笔迹图层（用于 destination-out 只影响笔迹，不影响背景/材质）
      strokesCanvas: null,
      strokesCtx: null,
      // right mouse button rotate with wheel
      _rmbActive: false,
      // toolbar attachment styles (fixed to viewport edges)
      leftStyle: { position: 'fixed' },
      rightStyle: { position: 'fixed' },
      bottomStyle: { position: 'fixed' },
      // fullscreen / hint state
      isDisplayFullscreen: false,
      showRotateHint: false,
      showRotateHoverHint: false,
      slowRotateActive: null,
  // strokes layer dirty flag: when true, we must rebuild strokes offscreen
  _strokesDirty: true,
  // focus/press timing to avoid accidental dots when window just gained focus
  _lastFocusAt: 0,
  _pressDownAt: 0,
  // one-shot: suppress first click after the window regains focus
  _suppressNextClickAfterFocus: false,
  // 数字拼图叠加
  puzzleOverlay: null, // HTMLCanvasElement
  puzzleOverlayBox: null, // {x,y,w,h} in logical coords
  puzzlePaletteSize: 6,
  puzzleLegendWidth: 120,
  // 油画风格参数
  puzzleMaxColors: 40,            // 不超过80，默认40
  puzzleMinRegionPx: 10,         // 小碎块最小像素，低于则合并
  puzzleLabelSmoothPasses: 2,     // 标签多数滤波次数（3x3）
  // 处理模式：server 优先由后端生成；client 为本地解析
  puzzleProcessMode: 'server',
      strokes: [], // store completed strokes
      currentStroke: null,
      drawing: false,
      // long-press to draw
  	  pressThreshold: 150, // ms; no delay before drawing starts
      longPressTimer: null,
      pressActive: false,
      pendingPoint: null,
      // remember the initial down point to start stroke from when movement starts before threshold
      pressStartPoint: null,
      _pressStartScreen: null,
      undoStack: [],
        redoStack: [],
  // one-shot guard to skip coverage expansion (used when snapping to 'see-all')
  _skipCoverageOnce: false,
      // HUD提示
      _hudVisible: false,
      _hudMessage: '',
      _hudTimer: null,
      // 眼保色切换
      eyeProtectActive: false,
      _prevBgColor: null,
      // 预置护眼色列表（常用）
      eyeProtectColors: [
        { name: '豆沙绿', hex: '#C7EDCC' },
        { name: '银河白', hex: '#FFFFFF' },
        { name: '杏仁黄', hex: '#FAF9DE' },
        { name: '秋叶褐', hex: '#FFF2E2' },
        { name: '胭脂红', hex: '#FDE6E0' },
        { name: '海天蓝', hex: '#DCE2F1' },
        { name: '葛巾紫', hex: '#E9EBFE' },
        { name: '极光灰', hex: '#EAEAEF' },
        { name: '青草绿', hex: '#E3EDCD' },
        { name: '电脑管家', hex: '#CCE8CF' },
        { name: 'WPS', hex: '#6E7B6C' }
      ],
      
      // brush color picker state: when user opens color picker we delay adding to recent until they actually draw
      _brushPickerOpen: false,
      _pendingBrushColor: null,
      _brushPickedPendingSave: false,
      // selected eye-protect color in the new dropdown ('' means off)
      _selectedEyeColor: '',
      // 最近使用的颜色（非黑白），用于画布和画笔的快速选择
      recentBgColors: [],
      recentBrushColors: [],
      // 粘贴预览/编辑
      pasteActive: false,
      pasteStroke: null, // {type:'image', src, _img, box:{x,y,w,h}, angle?:number, opacity?:number}
      _pasteDragging: false,
      _pasteDragStart: null,
      _pasteStartBox: null,
  // 句柄拖拽（缩放/旋转）
  _handleActive: null, // { kind:'scale'|'rotate', edge?:'n'|'s'|'e'|'w'|'nw'|'ne'|'sw'|'se' }
  _handleStart: null, // { p0:{x,y}, box0:{x,y,w,h}, angle0:number }
      // 编辑已提交的文本笔划（双击进入编辑）
      _editSourceIndex: null,
      _editSourceStroke: null,
    }
  },
  computed: {
    // scale used for drawing (auto cover fit * user scale)
    displayScale() {
      // keep view scale stable when rotating: use unrotated size for fit
      const w = this.width, h = this.height
      const fit = Math.max((this.viewW || 1) / (w || 1), (this.viewH || 1) / (h || 1))
      const s = fit * (this.scale || 1)
      // limit precision to avoid tiny float drift causing visual shifts between strokes
      return Number(s.toFixed(6))
    },
    onTextStyleChange(){
      try { this.invalidate(); this.saveAutosave() } catch(_){ }
    },
    // High-contrast hint text color based on current background
    hintTextColor() {
      return this.contrastingColor(this.bgColor)
    },
    canvasWrapStyle() {
      // no DOM transforms; drawing handles transforms
      return { background: 'transparent' }
    }
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext('2d')
  // ensure canvas fills the browser viewport
  this.resizeCanvas()
  window.addEventListener('resize', this.resizeCanvas)
    // toolbar positioning and cursor
    this.$nextTick(()=>{
      this.updateToolbarPositions()
      this.updateCursor()
    })
  // toolbars are fixed to viewport edges; initial position update
  this.updateToolbarPositions()
  window.addEventListener('resize', this.updateToolbarPositions)
    // keep toolbars positioned when scale or rotation change
    this.$watchers = []
    // watch properties manually
  // when transform props change, ensure toolbar placements stay correct
  this.$watchers.push(this.$watch('scale', ()=>{ this.updateToolbarPositions(); this.updateCursor() }))
  this.$watchers.push(this.$watch('rotation', ()=>{ this.updateToolbarPositions(); this.updateCursor() }))
  this.$watchers.push(this.$watch('width', this.updateToolbarPositions))
  this.$watchers.push(this.$watch('height', this.updateToolbarPositions))
    this.$watchers.push(this.$watch('brushSize', this.updateCursor))
    this.$watchers.push(this.$watch('brushStyle', this.updateCursor))
    this.$watchers.push(this.$watch('erasing', this.updateCursor))
  this.$watchers.push(this.$watch('eraserSize', this.updateCursor))
  document.addEventListener('fullscreenchange', this.onFullscreenChange)
    // 粘贴支持：监听系统粘贴
    window.addEventListener('paste', this.onPaste)
    // load autosave if present
    this.loadAutosave()
      // initialize undo stack with current state if empty (strip transient fields)
      if (!this.undoStack || !this.undoStack.length) {
        const serial = (this.strokes || []).map(s=>{ const { _img, ...rest } = s || {}; return rest })
        this.undoStack = [JSON.stringify(serial)]
      }
    // pointer events
    const c = this.$refs.canvas
  c.addEventListener('pointerdown', this.onPointerDown)
  c.addEventListener('pointermove', this.onPointerMove)
  c.addEventListener('pointerleave', this.onPointerLeave)
  c.addEventListener('dblclick', this.onCanvasDblClick)
  // prevent context menu to allow RMB + wheel interaction
  c.addEventListener('contextmenu', (e)=> e.preventDefault())
    window.addEventListener('pointerup', this.onPointerUp)
  // track window focus/blur to suppress unintended click-dots right after focus
  window.addEventListener('focus', this.onWindowFocus)
  window.addEventListener('blur', this.onWindowBlur)
    // CTRL + mouse wheel to zoom canvas (prevent browser page zoom)
    window.addEventListener('wheel', this.onCtrlWheel, { passive: false })
    // Space to temporarily enable hand (pan)
    window.addEventListener('keydown', this.onKeyDown, { passive: false })
    window.addEventListener('keyup', this.onKeyUp, { passive: false })
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.resizeCanvas)
    const c = this.$refs.canvas
    if (c) {
      c.removeEventListener('pointerdown', this.onPointerDown)
      c.removeEventListener('pointermove', this.onPointerMove)
      c.removeEventListener('pointerleave', this.onPointerLeave)
      c.removeEventListener('dblclick', this.onCanvasDblClick)
    }
    window.removeEventListener('pointerup', this.onPointerUp)
    window.removeEventListener('resize', this.updateToolbarPositions)
    document.removeEventListener('fullscreenchange', this.onFullscreenChange)
  window.removeEventListener('paste', this.onPaste)
    window.removeEventListener('wheel', this.onCtrlWheel)
  window.removeEventListener('keydown', this.onKeyDown)
  window.removeEventListener('keyup', this.onKeyUp)
    window.removeEventListener('focus', this.onWindowFocus)
    window.removeEventListener('blur', this.onWindowBlur)
    if (this.$watchers && this.$watchers.length) {
      for (const un of this.$watchers) { try { un() } catch(e){} }
    }
  },
  methods: {
    openImportPicker() {
      this.$refs.importFileInput?.click()
    },
    async importFile(file) {
      if (!file) return
      const buf = await file.arrayBuffer()
      return this.importBoardBuffer(buf)
    },
    async importBoardBuffer(buf) {
      const u8 = new Uint8Array(buf)
      let payload = null
      let importNotice = ''
      let unresolvedImageCount = 0
      const normalizePayload = (input)=>{
        const normalized = input && typeof input === 'object' ? input : {}
        const strokes = Array.isArray(normalized.strokes) ? normalized.strokes : []
        normalized.strokes = strokes.map((stroke)=>{
          if (!stroke || typeof stroke !== 'object') return stroke
          const { _img, ...rest } = stroke
          if (rest.type === 'image' && typeof rest.src === 'string' && rest.src.startsWith('blob:')) {
            unresolvedImageCount += 1
            return {
              ...rest,
              _importMissing: true,
              _missingReason: '旧版导出未内嵌图片资源'
            }
          }
          return rest
        })
        return normalized
      }
      const parseTextPayload = (source)=> {
        const text = new TextDecoder().decode(source).trim()
        return JSON.parse(text)
      }
      const isZip = u8.length >= 4 && u8[0] === 0x50 && u8[1] === 0x4B
      if (!isZip) {
        payload = parseTextPayload(buf)
      } else {
        try {
          const zip = await JSZip.loadAsync(buf)
          const jsonEntry = zip.file(/^(index|board|payload|whiteboard)\.json$/i)[0]
            || zip.file(/\.json$/i)[0]
          if (!jsonEntry) throw new Error('ZIP 内未找到清单 JSON')
          const text = await jsonEntry.async('string')
          payload = JSON.parse(text)
          const assetFiles = zip.file(/\.(png|jpg|jpeg|webp)$/i)
          const assetMap = {}
          for (const af of assetFiles) {
            const blob = await af.async('blob')
            const url = URL.createObjectURL(blob)
            const name = af.name.split('/').pop()
            assetMap[name] = url
            assetMap[af.name] = url
          }
          if (payload && payload.strokes && assetFiles.length) {
            for (const s of (payload.strokes || [])) {
              if (s && s.type === 'image' && s.src && typeof s.src === 'string') {
                const key = s.src.split('/').pop()
                if (assetMap[s.src]) s.src = assetMap[s.src]
                else if (assetMap[key]) s.src = assetMap[key]
              }
            }
          }
        } catch (_) {
          payload = parseTextPayload(buf)
          importNotice = '已按旧版 JSON 草稿格式导入'
        }
      }
      payload = normalizePayload(payload)
      if (!(payload && payload.meta)) throw new Error('无效的白板数据')

      this.width = payload.meta.width ?? this.width
      this.height = payload.meta.height ?? this.height
      this.bgColor = payload.meta.bgColor ?? this.bgColor
      this.texture = payload.meta.texture ?? this.texture
      this.rotation = payload.meta.rotation ?? this.rotation
      this.scale = payload.meta.scale ?? this.scale
      if (typeof payload.meta.panX === 'number') this.panX = payload.meta.panX
      if (typeof payload.meta.panY === 'number') this.panY = payload.meta.panY
      this.strokes = payload.strokes ?? []
      this.clampLogicalSize(true)
      this.resizeCanvas()
      this._strokesDirty = true
      this.redraw()
      try {
        requestAnimationFrame(()=>{ this._strokesDirty = true; this.redraw() })
        setTimeout(()=>{ this._strokesDirty = true; this.redraw() }, 300)
      } catch(_){ }
      try { this.saveAutosave() } catch(_){ }
      try {
        this.embedAllExternalImages(true).then(()=>{
          try { this._strokesDirty = true; this.redraw(); this.saveAutosave() } catch(_){ }
        })
      } catch(_){ }
      if (unresolvedImageCount > 0) {
        const msg = `已导入，但有 ${unresolvedImageCount} 张旧版 blob 图片无法恢复；旧文件没有内嵌图片资源。`
        try { this.showHUD(msg, 2200) } catch(_){ try { alert(msg) } catch(__){} }
      } else if (importNotice) {
        try { this.showHUD(importNotice, 1400) } catch(_){ }
      }
    },
    async onPaste(e){
      try{
        const dt = e.clipboardData || (e.originalEvent && e.originalEvent.clipboardData)
        if (!dt || !dt.items || !dt.items.length) return
        let file = null
        for (const it of dt.items) {
          if (it.kind === 'file' && it.type && it.type.startsWith('image/')) { file = it.getAsFile(); break }
        }
        if (file) {
          e.preventDefault()
          const url = URL.createObjectURL(file)
          // 初始尺寸：不超过逻辑画布的一半，保持比例
          const img = new Image()
          img.onload = ()=>{
            const maxW = Math.max(64, this.width * 0.5)
            const maxH = Math.max(64, this.height * 0.5)
            const iw = img.naturalWidth || img.width
            const ih = img.naturalHeight || img.height
            const s = Math.min(1, Math.min(maxW/iw, maxH/ih))
            const w = Math.max(2, Math.round(iw * s))
            const h = Math.max(2, Math.round(ih * s))
            const cx = this.width/2, cy = this.height/2
            this.pasteStroke = { type:'image', src:url, _img:img, box:{ x: cx - w/2, y: cy - h/2, w, h }, angle:0, opacity:1 }
            this.pasteActive = true
            this.invalidate()
            // 立刻保存一次，确保刷新后可恢复粘贴预览
            try { this.saveAutosave() } catch(_){ }
          }
          img.onerror = ()=>{ try{ URL.revokeObjectURL(url) }catch(_){} }
          img.src = url
          return
        }
        // 尝试作为纯文本粘贴
        let text = ''
        try { text = dt.getData && dt.getData('text/plain') || '' } catch(_){ }
        if (!text) {
          // 从 items 中异步获取字符串
          const strItem = Array.from(dt.items || []).find(it => it.kind === 'string' && /text\/plain/i.test(it.type||''))
          if (strItem && strItem.getAsString) {
            text = await new Promise(res => { try{ strItem.getAsString(t=>res(t||'')) }catch(_){ res('') } })
          }
        }
        if (text && text.trim()) {
          e.preventDefault()
          const lines = text.replace(/\r\n/g, '\n').split('\n')
          const base = Math.max(16, Math.round(Math.min(this.width, this.height) * 0.04))
          const c = document.createElement('canvas'); const mctx = c.getContext('2d')
          mctx.font = `${base}px sans-serif`
          const lineH = Math.round(base * 1.2)
          let natW = 0
          for (const ln of lines) { natW = Math.max(natW, Math.ceil(mctx.measureText(ln).width || 0)) }
          const natH = Math.max(lineH, lines.length * lineH)
          const maxW = Math.max(80, Math.round(this.width * 0.6))
          const maxH = Math.max(40, Math.round(this.height * 0.5))
          const w = Math.min(Math.max(40, natW), maxW)
          const h = Math.min(Math.max(24, natH), maxH)
          const cx = this.width/2, cy = this.height/2
          this.pasteStroke = {
            type: 'text',
            text,
            color: this.brushColor || '#000',
            fontFamily: 'sans-serif',
            fontWeight: 'normal',
            baseSize: base,
            align: 'left',
            lineHeightMult: 1.2,
            letterSpacing: 0,
            bgColor: '#00000000',
            textStrokeColor: '#000000',
            textStrokeWidth: 0,
            textShadowColor: '#000000',
            textShadowBlur: 0,
            textShadowOffsetX: 0,
            textShadowOffsetY: 0,
            fillBox: false,
            opacity: 1,
            angle: 0,
            box: { x: cx - w/2, y: cy - h/2, w, h }
          }
          this.pasteActive = true
          this.invalidate()
          try { this.saveAutosave() } catch(_){ }
          return
        }
      }catch(err){ /* ignore */ }
    },
    _commitPaste(){
      if (!this.pasteActive || !this.pasteStroke) return
      // 尝试把粘贴图片转换为 dataURL，避免刷新后 blob: 失效
      try{
        const s = this.pasteStroke
        if (s && s.type==='image' && s._img && s._img.complete && (s.src && typeof s.src==='string' && s.src.startsWith('blob:'))) {
          const img = s._img
          const c = document.createElement('canvas')
          c.width = img.naturalWidth || img.width
          c.height = img.naturalHeight || img.height
          const cctx = c.getContext('2d')
          cctx.drawImage(img, 0, 0)
          try { s.src = c.toDataURL('image/png') } catch(_){ /* ignore */ }
        }
      }catch(_){ }
      // 在修改笔划列表前记录撤销快照
      try { this.pushUndoSnapshot(); this.redoStack = [] } catch(_){}
      // 移入绘画层（若来自双击编辑，按原位置插回）
      if (this._editSourceIndex != null && this._editSourceIndex >= 0 && this._editSourceIndex <= this.strokes.length) {
        this.strokes.splice(this._editSourceIndex, 0, this.pasteStroke)
      } else {
        this.strokes.push(this.pasteStroke)
      }
      this._strokesDirty = true
      this.pasteActive = false
      this.pasteStroke = null
      // 清理编辑来源状态
      this._editSourceIndex = null
      this._editSourceStroke = null
      this.saveAutosave && this.saveAutosave()
      this.redraw()
    },
    _cancelPaste(){
      try {
        if (this.pasteStroke && this.pasteStroke.src && this.pasteStroke.src.startsWith('blob:')) {
          try{ URL.revokeObjectURL(this.pasteStroke.src) }catch(_){ }
        }
      }catch(_){ }
      // 如果是编辑已有文本，恢复原笔划
      if (this._editSourceIndex != null && this._editSourceStroke) {
        const idx = Math.min(Math.max(0, this._editSourceIndex), this.strokes.length)
        this.strokes.splice(idx, 0, this._editSourceStroke)
        this._strokesDirty = true
        this.saveAutosave && this.saveAutosave()
        this.redraw()
      }
      // 清理状态
      this.pasteActive = false
      this.pasteStroke = null
      this._editSourceIndex = null
      this._editSourceStroke = null
      this.invalidate()
    },
    // 画布双击：命中文本笔划则进入编辑模式
    onCanvasDblClick(e){
      try{
        if (this.pasteActive) return
        // 忽略在拖动画布/橡皮擦等模式下的双击
        if (this.isPanning || this.panMode) return
        const p = this.mapEventToCanvas(e)
        // 如上一击误触产生了一个点，且与双击位置几乎重合，则清理该点
        try {
          const last = this.strokes[this.strokes.length-1]
          if (last && last.dot && !last.eraser && last.points && last.points[0]) {
            const a = last.points[0]
            const dx = (a.x - p.x), dy = (a.y - p.y)
            if ((dx*dx + dy*dy) <= 25) { // 5px 内
              this.strokes.pop()
              this._strokesDirty = true
            }
          }
        } catch(_){}
        // 从上往下命中检测
        for (let i=this.strokes.length-1; i>=0; i--) {
          const s = this.strokes[i]
          if (!s || s.type !== 'text' || !s.box) continue
          if (this._hitTextStroke(p, s)) {
            // 进入编辑：移除该笔划，作为粘贴预览重启编辑
            this._editSourceIndex = i
            this._editSourceStroke = s
            this.pasteStroke = JSON.parse(JSON.stringify(s))
            this.pasteActive = true
            // 从笔迹层移除原笔划
            this.strokes.splice(i, 1)
            this._strokesDirty = true
            this.saveAutosave && this.saveAutosave()
            this.redraw()
            try { this.showHUD('文本编辑：回车确认，Esc取消') } catch(_){ }
            // 抑制紧随其后的 pointerup 触发的“点”
            this._suppressNextPointerUp = true
            return
          }
        }
      }catch(_){ }
    },
    _hitOverlayHandle(pt, s){
      try{
        const b = s && s.box
        if (!b) return null
        const sscale = Math.max(0.5, this.displayScale || 1)
        const tol = Math.max(6, Math.round(10 / sscale))
        const near = (x,y)=> Math.abs(pt.x - x) <= tol && Math.abs(pt.y - y) <= tol
        // rotation handle position (top center above box)
        const off = Math.max(18, Math.round(28 / sscale))
        const rx = b.x + b.w/2, ry = b.y - off
        if (Math.hypot(pt.x - rx, pt.y - ry) <= tol) return { kind: 'rotate' }
        // corners
        if (near(b.x, b.y)) return { kind:'scale', edge:'nw' }
        if (near(b.x + b.w, b.y)) return { kind:'scale', edge:'ne' }
        if (near(b.x, b.y + b.h)) return { kind:'scale', edge:'sw' }
        if (near(b.x + b.w, b.y + b.h)) return { kind:'scale', edge:'se' }
        // edges mid
        if (near(b.x + b.w/2, b.y)) return { kind:'scale', edge:'n' }
        if (near(b.x + b.w/2, b.y + b.h)) return { kind:'scale', edge:'s' }
        if (near(b.x, b.y + b.h/2)) return { kind:'scale', edge:'w' }
        if (near(b.x + b.w, b.y + b.h/2)) return { kind:'scale', edge:'e' }
        return null
      }catch(_){ return null }
    },
    _dragOverlayHandle(p){
      try{
        const H = this._handleActive
        if (!H || !this._handleStart || !this.pasteStroke) return
        const b = this.pasteStroke.box
        const b0 = this._handleStart.box0
        const p0 = this._handleStart.p0
        const dx = p.x - p0.x
        const dy = p.y - p0.y
        const clampW = (v)=> Math.max(2, Math.round(v))
        const clampH = (v)=> Math.max(2, Math.round(v))
        if (H.kind === 'rotate') {
          const cx = b0.x + b0.w/2, cy = b0.y + b0.h/2
          const a0 = Math.atan2(p0.y - cy, p0.x - cx)
          const a1 = Math.atan2(p.y - cy, p.x - cx)
          const delta = (a1 - a0) * 180/Math.PI
          this.pasteStroke.angle = (this._handleStart.angle0 + delta) % 360
          return
        }
        if (H.kind === 'scale') {
          const edge = H.edge
          const keepRatio = (edge === 'nw' || edge === 'ne' || edge === 'sw' || edge === 'se')
          const ratio = b0.w / Math.max(1e-6, b0.h)
          // start with base values
          let x = b0.x, y = b0.y, w = b0.w, h = b0.h
          const applyN = ()=>{ y = b0.y + dy; h = clampH(b0.h - dy) }
          const applyS = ()=>{ h = clampH(b0.h + dy) }
          const applyW = ()=>{ x = b0.x + dx; w = clampW(b0.w - dx) }
          const applyE = ()=>{ w = clampW(b0.w + dx) }
          if (edge === 'n') applyN()
          else if (edge === 's') applyS()
          else if (edge === 'w') applyW()
          else if (edge === 'e') applyE()
          else if (edge === 'nw') { applyN(); applyW() }
          else if (edge === 'ne') { applyN(); applyE() }
          else if (edge === 'sw') { applyS(); applyW() }
          else if (edge === 'se') { applyS(); applyE() }
          if (keepRatio) {
            // choose dominant change and enforce ratio
            const dw = w - b0.w, dh = h - b0.h
            if (Math.abs(dw/b0.w) >= Math.abs(dh/b0.h)) {
              // width dominant
              const newW = w
              const newH = clampH(newW / ratio)
              if (edge.includes('n')) { // adjust y so bottom anchored
                const bottom = b0.y + b0.h
                y = bottom - newH
              }
              h = newH
            } else {
              const newH = h
              const newW = clampW(newH * ratio)
              if (edge.includes('w')) { // adjust x so right anchored
                const right = b0.x + b0.w
                x = right - newW
              }
              w = newW
            }
          }
          b.x = x; b.y = y; b.w = w; b.h = h
        }
      }catch(_){ }
    },
    _hitTextStroke(pt, stroke){
      try{
        const b = stroke.box
        const ang = Number(stroke.angle||0)
        if (!ang) {
          return pt.x>=b.x && pt.x<=b.x+b.w && pt.y>=b.y && pt.y<=b.y+b.h
        }
        // 逆向旋转点到未旋转矩形坐标系
        const cx = b.x + b.w/2, cy = b.y + b.h/2
        const rad = -ang * Math.PI/180
        const cos = Math.cos(rad), sin = Math.sin(rad)
        const rx = cos*(pt.x - cx) - sin*(pt.y - cy) + cx
        const ry = sin*(pt.x - cx) + cos*(pt.y - cy) + cy
        return rx>=b.x && rx<=b.x+b.w && ry>=b.y && ry<=b.y+b.h
      }catch(_){ return false }
    },
    toggleDebugPanel(){ this.debugOpen = !this.debugOpen },
    onPuzzleFileChange(e){
      try{
        const f = e && e.target && e.target.files && e.target.files[0]
        if (!f) return
        // reset input so the same file can be selected again later
        e.target.value = ''
        if (this.puzzleProcessMode === 'server') {
          this._requestPuzzleServer(f)
          return
        }
        // fallback to client pipeline when explicitly chosen
        const url = URL.createObjectURL(f)
        const img = new Image()
        img.onload = ()=>{ try { URL.revokeObjectURL(url) } catch(_){ }; this.makeNumberPuzzle(img) }
        img.onerror = ()=>{ try { URL.revokeObjectURL(url) } catch(_){ } }
        img.src = url
      }catch(_){ }
    },
    _requestPuzzleServer(file){
      // 以表单上传到后端，期望返回处理后的PNG/SVG；这里先实现PNG图片路径
      const form = new FormData()
      form.append('file', file, file.name || 'image.png')
      // 可根据需要附带参数
      form.append('maxColors', String(Math.min(80, this.puzzleMaxColors||40)))
      form.append('minRegionPx', String(this.puzzleMinRegionPx||120))
      form.append('smoothPasses', String(this.puzzleLabelSmoothPasses||2))
      const endpoint = this.puzzleApiEndpoint || '/api/puzzle/paintbynumbers'
      this.showHUD && this.showHUD('正在由后端生成…')
      fetch(endpoint, { method: 'POST', body: form })
        .then(res=>{ if(!res.ok) throw new Error('HTTP '+res.status); return res.blob() })
        .then(blob=> new Promise((resolve)=>{ const url=URL.createObjectURL(blob); const img=new Image(); img.onload=()=>{ try{URL.revokeObjectURL(url)}catch(_){ }; resolve(img) }; img.onerror=()=>{ try{URL.revokeObjectURL(url)}catch(_){ }; resolve(null) }; img.src=url }))
        .then(img=>{
          if(!img){ this.showHUD && this.showHUD('后端返回无效图片'); return }
          this._insertImageStrokeFromImage(img)
          this.showHUD && this.showHUD('已插入后端结果')
        })
        .catch(err=>{ console.error('puzzle server failed', err); this.showHUD && this.showHUD('后端生成失败，请检查服务') })
    },
    _insertImageStrokeFromImage(img){
      // 居中放置到逻辑画布中，保持比例
      const iw = img.naturalWidth || img.width || 1
      const ih = img.naturalHeight || img.height || 1
      const fitS = Math.min(this.width / iw, this.height / ih, 1)
      const dw = Math.round(iw * fitS)
      const dh = Math.round(ih * fitS)
      const bx = Math.floor((this.width - dw)/2)
      const by = Math.floor((this.height - dh)/2)
      try{
        // 为保证可保存，转为dataURL
        const c = document.createElement('canvas'); c.width=iw; c.height=ih; const cctx=c.getContext('2d'); cctx.drawImage(img,0,0)
        const dataURL = c.toDataURL('image/png')
        const imgStroke = { type:'image', box:{x:bx,y:by,w:dw,h:dh}, src:dataURL, opacity:1 }
        this.pushUndoSnapshot(); this.redoStack=[]; this.strokes.push(imgStroke); this._strokesDirty=true; this.redraw(); this.saveAutosave()
      }catch(e){ console.error('insert image stroke failed', e) }
    },
    makeNumberPuzzle(img){
      // 1) 限制尺寸，拷入离屏
      const maxSide = 768
      let w = img.naturalWidth || img.width
      let h = img.naturalHeight || img.height
      const scale = Math.min(1, maxSide / Math.max(1, Math.max(w,h)))
      w = Math.max(1, Math.round(w * scale))
      h = Math.max(1, Math.round(h * scale))
      const src = document.createElement('canvas')
      src.width = w; src.height = h
      const sctx = src.getContext('2d')
      sctx.drawImage(img, 0, 0, w, h)
      const imgData = sctx.getImageData(0,0,w,h)
      // 2) 颜色聚类（k-means）
  // 2.1) 量化到有限颜色（先粗化，最多80色）
  const kWanted = Math.max(4, Math.round(this.puzzlePaletteSize || 12))
  const k = Math.min(80, Math.max(4, Math.min(this.puzzleMaxColors || 40, kWanted)))
  let { labels, centers } = this._kmeansQuantize(imgData.data, w, h, k, 8)
  // 2.2) 对标签做多数滤波，去除椒盐噪声，保边粗化
  labels = this._modeFilterLabels(labels, w, h, this.puzzleLabelSmoothPasses || 2)
  // 2.3) 合并小碎块到相邻最大接触面标签
  const minPx = Math.max(24, this.puzzleMinRegionPx || 120)
  labels = this._mergeSmallRegions(labels, w, h, minPx)
  // 2.4) 压缩标签并用原始像素重新估计调色板颜色
  const remapInfo = this._remapLabelsAndPalette(labels, w, h, imgData.data)
  labels = remapInfo.labels
  centers = remapInfo.colors
  // 3) 连通域（用于编号）
  const comps = this._connectedComponents(labels, w, h)
      // 4) 生成黑白线稿 + 数字 + 左侧色卡
      const legendW = this.puzzleLegendWidth
      const ss = Math.max(2, Math.ceil(this._dpr || 2)) // 超采样倍数
      const out = document.createElement('canvas')
      out.width = (w + legendW) * ss
      out.height = h * ss
      const octx = out.getContext('2d')
      octx.imageSmoothingEnabled = true
      try{ octx.imageSmoothingQuality = 'high' }catch(_){ }
      // 背景白
      octx.fillStyle = '#ffffff'; octx.fillRect(0,0,out.width,out.height)
      // 轮廓线：相邻不同标签边界画线
      octx.save(); octx.translate(legendW * ss, 0)
      octx.strokeStyle = '#000'; octx.globalAlpha = 0.95; octx.lineWidth = Math.max(1, 1.2 * ss); octx.lineJoin = 'round'; octx.lineCap = 'round'; octx.miterLimit = 2
      this._drawOutlines(octx, labels, w, h, ss)
      // 在每个连通域中心写数字（标签+1）
      octx.fillStyle = '#000'; octx.font = `bold ${Math.round(12*ss)}px sans-serif`; octx.textAlign = 'center'; octx.textBaseline = 'middle'
      for(const c of comps){
        if (c.area < 40) continue // 小块忽略编号
        octx.fillText(String(c.label+1), c.cx * ss, c.cy * ss)
      }
      octx.restore()
      // 左侧色卡及编号
      const sw = Math.min(legendW-24, 64), sh = 18
      const sx = Math.floor((legendW - (sw + 16)) / 2)
      octx.fillStyle = '#111'; octx.font = `${Math.round(12*ss)}px sans-serif`; octx.textAlign = 'left'; octx.textBaseline = 'middle'
      for(let i=0;i<centers.length;i++){
        const cy = 14 + i*(sh+6)
        const col = `rgb(${Math.round(centers[i][0])},${Math.round(centers[i][1])},${Math.round(centers[i][2])})`
        octx.fillStyle = col; octx.fillRect(sx*ss, (cy - sh/2)*ss, sw*ss, sh*ss)
        octx.strokeStyle = '#000'; octx.lineWidth = Math.max(1, 1*ss); octx.strokeRect(sx*ss, (cy - sh/2)*ss, sw*ss, sh*ss)
        octx.fillStyle = '#000'; octx.fillText(String(i+1), (sx + sw + 8)*ss, cy*ss)
      }
      // 5) 放到画布中央（逻辑坐标）
      const fitS = Math.min(this.width / out.width, this.height / out.height, 1)
      const dw = Math.round(out.width * fitS)
      const dh = Math.round(out.height * fitS)
      const bx = Math.floor((this.width - dw)/2)
      const by = Math.floor((this.height - dh)/2)
      // 将生成结果插入为可保存的“图像笔划”
      try {
        const dataURL = out.toDataURL('image/png')
        const imgStroke = {
          type: 'image',
          box: { x: bx, y: by, w: dw, h: dh },
          src: dataURL,
          opacity: 1
        }
        // 记录撤销快照
        this.pushUndoSnapshot()
        this.redoStack = []
        this.strokes.push(imgStroke)
        this._strokesDirty = true
        this.redraw()
        this.saveAutosave()
      } catch (e) {
        console.error('embed puzzle as stroke failed', e)
        // 回退到旧的覆盖显示方案
        this.puzzleOverlay = out
        this.puzzleOverlayBox = { x: bx, y: by, w: dw, h: dh }
      }
      this.showHUD && this.showHUD('已生成数字拼图')
      this.redraw()
    },
    _kmeansQuantize(data, w, h, k=6, iters=8){
      // K-Means++ 初始化，避免背景单色主导导致所有聚到同一类
      const N = w*h
      const labels = new Uint16Array(N)
      const centers = []
      const dist2=(r,g,b,c)=>{ const dr=r-c[0], dg=g-c[1], db=b-c[2]; return dr*dr+dg*dg+db*db }
      // choose first center as a random pixel with stride to avoid bias
      const first = Math.floor(Math.random()*N)
      centers.push([data[first*4], data[first*4+1], data[first*4+2]])
      // precompute min distance squared to current centers for each pixel
      let minD2 = new Float64Array(N)
      for(let i=0,p=0;i<N;i++,p+=4){ const r=data[p],g=data[p+1],b=data[p+2]; minD2[i]=dist2(r,g,b,centers[0]) }
      while(centers.length < k){
        // pick next center with probability proportional to minD2
        let sum=0; for(let i=0;i<N;i++) sum += minD2[i]
        if (sum<=0){
          // all identical colors; duplicate last center to keep k length
          const c = centers[centers.length-1]; centers.push([c[0],c[1],c[2]]); continue
        }
        let r = Math.random()*sum, acc=0, chosen=0
        for(let i=0;i<N;i++){ acc += minD2[i]; if(acc>=r){ chosen=i; break } }
        centers.push([data[chosen*4], data[chosen*4+1], data[chosen*4+2]])
        // update minD2
        const cNew = centers[centers.length-1]
        for(let i=0,p=0;i<N;i++,p+=4){ const d=dist2(data[p],data[p+1],data[p+2],cNew); if(d<minD2[i]) minD2[i]=d }
      }
      // standard k-means iterations
      for(let it=0; it<iters; it++){
        // assign
        for(let i=0, p=0; i<N; i++, p+=4){
          const r=data[p], g=data[p+1], b=data[p+2]
          let bi=0, bd=Infinity
          for(let j=0;j<k;j++){ const d=dist2(r,g,b,centers[j]); if(d<bd){bd=d; bi=j} }
          labels[i]=bi
        }
        // update
        const sum=new Array(k).fill(0).map(()=>[0,0,0,0])
        for(let i=0, p=0; i<N; i++, p+=4){ const j=labels[i]; sum[j][0]+=data[p]; sum[j][1]+=data[p+1]; sum[j][2]+=data[p+2]; sum[j][3]++ }
        for(let j=0;j<k;j++){
          if(sum[j][3]>0){ centers[j][0]=sum[j][0]/sum[j][3]; centers[j][1]=sum[j][1]/sum[j][3]; centers[j][2]=sum[j][2]/sum[j][3] }
        }
      }
      return { labels, centers }
    },
    // 多数滤波：对量化后的标签做 3x3 模式滤波，重复 passes 次
  _modeFilterLabels(labels, w, h, passes=2){
      if (!passes || passes<=0) return labels
      let src = labels, dst = new Uint16Array(w*h)
      const idx=(x,y)=>y*w+x
      for(let pass=0; pass<passes; pass++){
        for(let y=0;y<h;y++){
          for(let x=0;x<w;x++){
            const hist = new Map()
            for(let dy=-1;dy<=1;dy++){
              const yy=y+dy; if(yy<0||yy>=h) continue
              for(let dx=-1;dx<=1;dx++){
                const xx=x+dx; if(xx<0||xx>=w) continue
                const lab = src[idx(xx,yy)]
                hist.set(lab, (hist.get(lab)||0)+1)
              }
            }
            // choose label with max count, tie -> center pixel label
            let best=src[idx(x,y)], bc=-1
            hist.forEach((v,k)=>{ if(v>bc){ bc=v; best=k } })
            dst[idx(x,y)] = best
          }
        }
        // swap
        if (pass < passes-1){ const t=src; src=dst; dst=t }
      }
      // 结果已在 dst（最后一轮未交换）
      return dst
    },
    // 合并面积小于 minAreaPx 的连通域：将其像素改成接触最多的邻接标签
    _mergeSmallRegions(labels, w, h, minAreaPx=120){
      const idx=(x,y)=>y*w+x
      const N=w*h
      const out = new Uint16Array(labels) // work copy
      let changed=true, guard=0
      while(changed && guard<6){
        changed=false; guard++
        // find components on current labels
        const comps = this._connectedComponents(out, w, h)
        for(const c of comps){
          if (c.area >= minAreaPx) continue
          // choose neighbor label with max border contacts
          const targetHist = new Map()
          // scan pixels of this component
          const lab = c.label
          // flood again but limited to this component; we can scan neighborhood by perimeter sampling
          // Brute force: iterate image and check pixels with this label near component centroid within square radius
          // Simpler and safe: walk all pixels and accumulate for 4-neigh borders
          for(let y=0;y<h;y++){
            for(let x=0;x<w;x++){
              const i=idx(x,y)
              if(out[i]!==lab) continue
              // right
              if(x+1<w && out[idx(x+1,y)]!==lab){ const nb=out[idx(x+1,y)]; targetHist.set(nb,(targetHist.get(nb)||0)+1) }
              // left
              if(x-1>=0 && out[idx(x-1,y)]!==lab){ const nb=out[idx(x-1,y)]; targetHist.set(nb,(targetHist.get(nb)||0)+1) }
              // down
              if(y+1<h && out[idx(x,y+1)]!==lab){ const nb=out[idx(x,y+1)]; targetHist.set(nb,(targetHist.get(nb)||0)+1) }
              // up
              if(y-1>=0 && out[idx(x,y-1)]!==lab){ const nb=out[idx(x,y-1)]; targetHist.set(nb,(targetHist.get(nb)||0)+1) }
            }
          }
          if (targetHist.size===0) continue
          let bestLab=lab, bestCnt=-1
          targetHist.forEach((v,k)=>{ if(v>bestCnt){ bestCnt=v; bestLab=k } })
          if (bestLab===lab) continue
          // relabel this component pixels to bestLab
          for(let y=0;y<h;y++){
            for(let x=0;x<w;x++){
              const i=idx(x,y); if(out[i]===lab) out[i]=bestLab
            }
          }
          changed=true
        }
      }
      return out
    },
    // 将离散标签压缩为 0..M-1，并按新标签统计原始像素均值作为色板
    _remapLabelsAndPalette(labels, w, h, data){
      const N=w*h
      const used = new Map()
      for(let i=0;i<N;i++){ used.set(labels[i], true) }
      const keys = Array.from(used.keys())
      keys.sort((a,b)=>a-b)
      const map = new Map(); keys.forEach((k,i)=>map.set(k,i))
      const out = new Uint16Array(N)
      const sumR=new Float64Array(keys.length), sumG=new Float64Array(keys.length), sumB=new Float64Array(keys.length), cnt=new Uint32Array(keys.length)
      for(let i=0,p=0;i<N;i++,p+=4){ const ni=map.get(labels[i]); out[i]=ni; sumR[ni]+=data[p]; sumG[ni]+=data[p+1]; sumB[ni]+=data[p+2]; cnt[ni]++ }
      const colors=[]
      for(let i=0;i<keys.length;i++){ const n=Math.max(1,cnt[i]); colors.push([sumR[i]/n, sumG[i]/n, sumB[i]/n]) }
      return { labels: out, colors }
    },
  _connectedComponents(labels, w, h){
      const seen = new Uint8Array(w*h) // 0:unseen, 1:queued, 2:visited
      const res = []
      const stackX = new Int32Array(w*h)
      const stackY = new Int32Array(w*h)
      for (let y=0; y<h; y++){
        for (let x=0; x<w; x++){
          const i = y*w + x
          if (seen[i]) continue
          const lab = labels[i]
          // start flood fill (LIFO stack is fine)
          let top = 0
          stackX[top] = x; stackY[top] = y; top++
          seen[i] = 1
          let area = 0, sx = 0, sy = 0
          while (top > 0){
            const cx = stackX[--top]
            const cy = stackY[top]
            const ii = cy*w + cx
            if (seen[ii] === 2) continue
            if (labels[ii] !== lab) continue
            seen[ii] = 2
            area++; sx += cx; sy += cy
            // neighbors 4-dir
            // right
            if (cx+1 < w){ const ni = cy*w + (cx+1); if (!seen[ni] && labels[ni]===lab){ seen[ni]=1; stackX[top]=cx+1; stackY[top]=cy; top++ } }
            // left
            if (cx-1 >= 0){ const ni = cy*w + (cx-1); if (!seen[ni] && labels[ni]===lab){ seen[ni]=1; stackX[top]=cx-1; stackY[top]=cy; top++ } }
            // down
            if (cy+1 < h){ const ni = (cy+1)*w + cx; if (!seen[ni] && labels[ni]===lab){ seen[ni]=1; stackX[top]=cx; stackY[top]=cy+1; top++ } }
            // up
            if (cy-1 >= 0){ const ni = (cy-1)*w + cx; if (!seen[ni] && labels[ni]===lab){ seen[ni]=1; stackX[top]=cx; stackY[top]=cy-1; top++ } }
          }
          if (area > 0){
            res.push({ label: lab, area, cx: sx/area, cy: sy/area })
          }
        }
      }
      return res
    },
    _drawOutlines(ctx, labels, w, h, s=1){
      const idx=(x,y)=> y*w+x
      ctx.beginPath()
      for(let y=0;y<h;y++){
        for(let x=0;x<w;x++){
          const i=idx(x,y), lab=labels[i]
          // right edge
          if (x===w-1 || labels[idx(x+1,y)]!==lab) { ctx.moveTo((x+1+0.5)*s, (y+0)*s); ctx.lineTo((x+1+0.5)*s, (y+1)*s) }
          // bottom edge
          if (y===h-1 || labels[idx(x,y+1)]!==lab) { ctx.moveTo((x+0)*s, (y+1+0.5)*s); ctx.lineTo((x+1)*s, (y+1+0.5)*s) }
        }
      }
      ctx.stroke()
    },
  onWindowFocus(){
    try{ this._lastFocusAt = (performance && performance.now) ? performance.now() : Date.now() }catch(_){ this._lastFocusAt = Date.now() }
    // suppress the very next click after we regain focus
    this._suppressNextClickAfterFocus = true
  },
  onWindowBlur(){
    // on blur, reset the marker so next focus can suppress one click
    this._suppressNextClickAfterFocus = true
  },
    saveDebugPreset(){
      try{
        const key = 'wb_debug_preset_v1'
        const data = JSON.stringify(this.dbg || {})
        localStorage.setItem(key, data)
        try{ this.showHUD && this.showHUD('已保存预设') }catch(_){ }
      }catch(e){ /* ignore */ }
    },
    loadDebugPreset(){
      try{
        const key = 'wb_debug_preset_v1'
        const raw = localStorage.getItem(key)
        if (!raw) { try{ this.showHUD && this.showHUD('未找到预设') }catch(_){ }; return }
        const val = JSON.parse(raw)
        if (val && typeof val==='object') {
          this.dbg = { ...(this.dbg||{}), ...val }
          // 触发一次重绘以便 seam 设置立即可见
          this.redraw && this.redraw()
          try{ this.showHUD && this.showHUD('已恢复预设') }catch(_){ }
        }
      }catch(e){ /* ignore */ }
    },
    invalidate(){
      if (this._rafPending) return
      this._rafPending = true
      requestAnimationFrame(()=>{ this._rafPending = false; try{ this.redraw() }catch(_){}})
    },
    computeFit(){
      const w = this.width || 1
      const h = this.height || 1
      const vw = this.viewW || 1
      const vh = this.viewH || 1
      // cover-fit: use max ratio so content可覆盖视口
      return Math.max(vw / w, vh / h)
    },
    // contain-fit: use min ratio so entire logical canvas fits in viewport (used as min zoom bound)
    computeContain(){
      const w = this.width || 1
      const h = this.height || 1
      const vw = this.viewW || 1
      const vh = this.viewH || 1
      return Math.min(vw / w, vh / h)
    },
    // contain for rotated content: fits the rotated bounding box into the viewport
    computeContainRotated(){
      const w = this.width || 1
      const h = this.height || 1
      const vw = this.viewW || 1
      const vh = this.viewH || 1
      const rad = Math.abs(((this.rotation % 360) + 360) % 360) * Math.PI / 180
      const cos = Math.abs(Math.cos(rad))
      const sin = Math.abs(Math.sin(rad))
      const rw = w * cos + h * sin // rotated bounding width in logical units
      const rh = w * sin + h * cos // rotated bounding height in logical units
      return Math.min(vw / (rw || 1), vh / (rh || 1))
    },
    clampLogicalSize(adjustPan = false) {
      try {
        const maxW = this._maxLogicalW || Infinity
        const maxH = this._maxLogicalH || Infinity
        let changed = false
        if (this.width > maxW) {
          const oldW = this.width
          this.width = maxW
          if (adjustPan) this.panX -= (oldW - this.width) / 2
          changed = true
        }
        if (this.height > maxH) {
          const oldH = this.height
          this.height = maxH
          if (adjustPan) this.panY -= (oldH - this.height) / 2
          changed = true
        }
        if (changed) { this.ensureStrokesLayer(); this.redraw() }
      } catch(e){}
    },
    // HUD 显示 helper
    showHUD(msg, ms = 900) {
      try { clearTimeout(this._hudTimer) } catch(_){}
      this._hudMessage = msg
      this._hudVisible = true
      this._hudTimer = setTimeout(()=>{ this._hudVisible = false; this._hudMessage = '' }, ms)
    },

    // color helpers for palettes
    setBgColor(color) {
      if (!color) return
      // 不再记录画布背景颜色到 recentBgColors（仅保留画笔最近色）
      this.bgColor = color
      this.updateCursor()
    },
    setBrushColor(color, record = true) {
      if (!color) return
      // when called programmatically (swatch clicks) record immediately
      this.brushColor = color
      if (record) {
        if (color.toLowerCase() !== '#000000' && color.toLowerCase() !== '#ffffff') {
          this.recentBrushColors = [color].concat(this.recentBrushColors.filter(c => c !== color)).slice(0,5)
        }
      }
      this.updateCursor()
    },
    onBrushColorChange(e) {
      const c = this.brushColor
      // when color picker is open, do not immediately record to recent; mark pending save
      if (this._brushPickerOpen) {
        this._pendingBrushColor = c
        this._brushPickedPendingSave = true
        // update preview but don't record
        this.setBrushColor(c, false)
      } else {
        // normal path (not from the popup) — record immediately
        this.setBrushColor(c, true)
      }
    },

    onBrushPickerOpen() {
      this._brushPickerOpen = true
      this._pendingBrushColor = null
      this._brushPickedPendingSave = false
    },

    onBrushPickerClose() {
      // color picker closed; keep the chosen color as current but wait until user actually draws to record it
      this._brushPickerOpen = false
      // pending color already set in onBrushColorChange
    },

    // When a color was chosen from the picker, only add it to recent after the user actually starts drawing
    commitPendingBrushColor() {
      if (this._brushPickedPendingSave && this._pendingBrushColor) {
        const color = this._pendingBrushColor
        if (color.toLowerCase() !== '#000000' && color.toLowerCase() !== '#ffffff') {
          this.recentBrushColors = [color].concat(this.recentBrushColors.filter(c => c !== color)).slice(0,5)
        }
      }
      this._pendingBrushColor = null
      this._brushPickedPendingSave = false
    },

    onEyeSelectChange() {
      const hex = this._selectedEyeColor
      if (!hex) {
        // 关闭护眼：背景统一改为白色
        this.eyeProtectActive = false
        this._selectedEyeColor = ''
        this.bgColor = '#ffffff'
        try { this.redraw() } catch(_){}
        return
      }
      // select a named eye-protect color
      this.selectEyeColor(hex)
    },
    onBgColorChange(e) {
      const c = this.bgColor
      this.setBgColor(c)
      try { this.redraw() } catch(_){}
      try { this.saveAutosave() } catch(_){}
      // 互相联动：颜色选择器变更 -> 同步护眼开关与选项
      try {
        const hex = (c || '').toLowerCase()
        const found = (this.eyeProtectColors||[]).find(x => (x.hex||'').toLowerCase() === hex)
        if (found) {
          this.eyeProtectActive = true
          this._selectedEyeColor = found.hex
        } else {
          this.eyeProtectActive = false
          this._selectedEyeColor = ''
        }
      } catch(_){ }
    },
    toggleEyeProtect() {
      const eye = '#f6f3e6'
      if (!this.eyeProtectActive) {
        this._prevBgColor = this.bgColor
        // set to first eyeProtect color by default
        const def = (this.eyeProtectColors && this.eyeProtectColors.length) ? this.eyeProtectColors[0].hex : eye
        this.bgColor = def
        this.eyeProtectActive = true
      } else {
        // 关闭护眼：背景统一改为白色
        this.eyeProtectActive = false
        this._selectedEyeColor = ''
        this.bgColor = '#ffffff'
      }
      this.updateCursor()
      // immediately repaint background
      try { this.redraw() } catch(_){}
      try { this.saveAutosave() } catch(_){}
    },

    selectEyeColor(hex) {
      if (!hex) return
      this._prevBgColor = this.bgColor
      this.bgColor = hex
      this.eyeProtectActive = true
      this._selectedEyeColor = hex
      this.updateCursor()
      try { this.redraw() } catch(_){}
      try { this.saveAutosave() } catch(_){}
    },
    // keyboard helpers
    isTypingTarget(el){
      if (!el) return false
      const tag = (el.tagName||'').toLowerCase()
      if (tag === 'input' || tag === 'textarea' || tag === 'select') return true
      if (el.isContentEditable) return true
      return false
    },
    onKeyDown(e){
      // 粘贴预览期的快捷键：回车确认、ESC取消、方向键调整
      if (this.pasteActive && this.pasteStroke) {
        const st = this.pasteStroke
        const b = st.box
        const stepMove = e.shiftKey ? 8 : 2
        const stepScale = e.shiftKey ? 0.08 : 0.03
        const stepRot = e.shiftKey ? 5 : 2
        const k = e.key || e.code || ''
        // Enter: 确认
        if (k === 'Enter' || k === 'NumpadEnter') {
          try { e.preventDefault() } catch(_){ }
          this._commitPaste()
          return
        }
        // Escape: 取消
        if (k === 'Escape') {
          try { e.preventDefault() } catch(_){ }
          this._cancelPaste()
          return
        }
        // 箭头键：移动 / 缩放 / 旋转
        const kc = (e.code||'')
        const ctrl = e.ctrlKey || e.metaKey
        const alt = e.altKey
        if (kc.startsWith('Arrow')) {
          try { e.preventDefault() } catch(_){ }
          if (alt) {
            // 旋转：左右旋转，上下等比缩放
            if (kc === 'ArrowLeft') st.angle = (st.angle||0) - stepRot
            else if (kc === 'ArrowRight') st.angle = (st.angle||0) + stepRot
            else if (kc === 'ArrowUp') { b.w *= (1+stepScale); b.h *= (1+stepScale) }
            else if (kc === 'ArrowDown') { b.w *= (1-stepScale); b.h *= (1-stepScale) }
          } else if (ctrl) {
            // 缩放：左右宽度，上下高度
            if (kc === 'ArrowLeft') b.w = Math.max(2, b.w * (1 - stepScale))
            else if (kc === 'ArrowRight') b.w = Math.max(2, b.w * (1 + stepScale))
            else if (kc === 'ArrowUp') b.h = Math.max(2, b.h * (1 + stepScale))
            else if (kc === 'ArrowDown') b.h = Math.max(2, b.h * (1 - stepScale))
          } else {
            // 移动
            if (kc === 'ArrowLeft') b.x -= stepMove
            else if (kc === 'ArrowRight') b.x += stepMove
            else if (kc === 'ArrowUp') b.y -= stepMove
            else if (kc === 'ArrowDown') b.y += stepMove
          }
          this.invalidate()
          return
        }
      }
      // ESC 一律作为退出稿纸（除非正在编辑文本输入）
      if (e.code === 'Escape' || e.key === 'Escape') {
        if (this.isTypingTarget(e.target)) return
        try { e.preventDefault() } catch(_){}
        this.exitWhiteboard()
        return
      }
      // Undo / Redo shortcuts
      // - Ctrl+Z: Undo
      // - Ctrl+Shift+Z: Redo
      try {
        const isTyping = this.isTypingTarget(e.target)
        if (!isTyping && (e.ctrlKey || e.metaKey)) {
          const k = (e.key || '').toLowerCase()
          if (k === 'z' && !e.shiftKey) {
            e.preventDefault()
            this.undo()
            return
          }
          if (k === 'z' && e.shiftKey) {
            e.preventDefault()
            this.redo()
            return
          }
        }
      } catch(_){}
  if (e.code === 'Space' || e.key === ' ') {
        // 如果当前焦点在 range 控件上，则不要把空格作为临时橡皮擦的开关
        const active = document.activeElement
        if (active && active.tagName && active.tagName.toLowerCase() === 'input' && active.type === 'range') return
        if (this.isTypingTarget(e.target)) return
        e.preventDefault()
        if (this._spaceEraserActive) return
        this._spaceEraserActive = true
        // 记录先前状态并切换为橡皮擦
        this._preErasing = this.erasing
        this._preBrushStyle = this.brushStyle
        this._prePanMode = this.panMode
        this.panMode = false
        this.erasing = true
        this.updateCursor()
        // HUD 提示：按住空格为临时橡皮擦
        this.showHUD('按住 空格：临时橡皮擦（滑轮调整大小）', 1000)
      }
      // HUD 提示：按住 Ctrl 可调整画笔大小（仅在按下 Ctrl 时显示）
      if (e.key === 'Control' || e.code === 'ControlLeft' || e.code === 'ControlRight') {
        this.showHUD('按住 Ctrl：滑轮调整画笔大小', 1000)
      }
    },
    onKeyUp(e){
      if (this.pasteActive) return
      if (e.code === 'Space' || e.key === ' ') {
          // 如果当前焦点在 range 控件上，则忽略 keyup
          const active = document.activeElement
          if (active && active.tagName && active.tagName.toLowerCase() === 'input' && active.type === 'range') return
          if (!this._spaceEraserActive) return
          e.preventDefault()
          this._spaceEraserActive = false
          // 恢复原状态
          this.erasing = this._preErasing
          this.brushStyle = this._preBrushStyle
          this.panMode = this._prePanMode
          if (!this.isPanning) this.updateCursor()
        }
    },
    togglePanMode(){ this.panMode = !this.panMode; if(this.panMode){ this.erasing=false; } this.updateCursor() },
    onPanHover(_) { /* reserved for future hover hint if needed */ },
    // Exit whiteboard: go back to previous page; fallback to home if no history
    exitWhiteboard() {
      this.$emit('request-exit')
    },
    // Handle Ctrl + mouse wheel as zoom in/out for the canvas
    onCtrlWheel(e) {
      // ignore when using slow-rotate wheel interaction (handled elsewhere)
      if (this.slowRotateActive) return
      // RMB + wheel: rotate about pointer as pivot
      if (this._rmbActive) {
        e.preventDefault()
        const delta = -e.deltaY * 0.15 // degrees
        this.rotateAroundPointer(delta, e)
        return
      }
      // Space 按住时：滚轮调节橡皮大小（优先）
      if (this._spaceEraserActive) {
        e.preventDefault()
        const step = (e.shiftKey ? 5 : 2)
        if (e.deltaY > 0) this.eraserSize = Math.max(2, this.eraserSize - step)
        else this.eraserSize = Math.min(32, this.eraserSize + step)
        this.updateCursor()
        return
      }
      // Ctrl + wheel: 改变当前画笔大小
      if (e.ctrlKey) {
        e.preventDefault()
        const step = (e.shiftKey ? 5 : 1)
        if (e.deltaY > 0) this.brushSize = Math.max(1, this.brushSize - step)
        else this.brushSize = Math.min(14, this.brushSize + step)
        this.updateCursor()
        return
      }
      // 普通滚轮（无 Ctrl）：缩放画布（保持先前的全局拦截行为）
      e.preventDefault()
      if (e.deltaY > 0) this.zoomOut(); else this.zoomIn()
    },
    rotateAroundPointer(deltaDeg, wheelEvent){
      try {
        // pivot in logical coords from current pointer position
        const pivot = this.mapEventToCanvas(wheelEvent)
        const v = { x: pivot.x - this.width/2, y: pivot.y - this.height/2 }
        // update pan so that pivot stays under the cursor after rotation
        const rad = (deltaDeg * Math.PI / 180)
        const cos = Math.cos(-rad), sin = Math.sin(-rad)
        const vx = v.x + this.panX
        const vy = v.y + this.panY
        const rx = vx * cos - vy * sin
        const ry = vx * sin + vy * cos
        this.panX = rx - v.x
        this.panY = ry - v.y
        // finally, update rotation
        this.rotation = (this.rotation + deltaDeg + 360) % 360
        this.redraw(); this.updateToolbarPositions()
      } catch(err) { /* ignore */ }
    },
    // choose black/white-ish text for best contrast with given bg color
    contrastingColor(hex) {
      try {
        let c = (hex || '').trim()
        if (!c) return '#111827'
        if (c.startsWith('#')) c = c.slice(1)
        if (c.length === 3) c = c.split('').map(ch => ch + ch).join('')
        const r = parseInt(c.slice(0,2), 16) / 255
        const g = parseInt(c.slice(2,4), 16) / 255
        const b = parseInt(c.slice(4,6), 16) / 255
        const toLinear = (u)=> (u <= 0.03928 ? u/12.92 : Math.pow((u+0.055)/1.055, 2.4))
        const R = toLinear(r), G = toLinear(g), B = toLinear(b)
        const L = 0.2126*R + 0.7152*G + 0.0722*B
        // threshold ~0.5 for good contrast; return near-black or near-white
        return L > 0.5 ? '#111827' : '#F9FAFB'
      } catch(e) { return '#111827' }
    },
    resizeCanvas() {
      // update viewport canvas pixels to fill browser
      const w = Math.max(300, Math.round(window.innerWidth))
      const h = Math.max(200, Math.round(window.innerHeight))
      this.viewW = w
      this.viewH = h

      // If auto-fit is enabled, make the logical board size N times the viewport so
      // the initial view only shows a portion of the board.
      if (this._autoFitContent) {
        const mul = this._autoFitLogicalMultiplier || 1
        this.width = Math.max(1, Math.round(this.viewW * mul))
        this.height = Math.max(1, Math.round(this.viewH * mul))
      }

      // set canvas backing size to match view size * DPR for sharp rendering
      try {
        this._dpr = (window.devicePixelRatio || 1)
        // recompute logical caps based on DPR and backing pixel cap
        this._maxLogicalW = Math.max(1024, Math.floor(this._maxBackingPixels / this._dpr))
        this._maxLogicalH = Math.max(1024, Math.floor(this._maxBackingPixels / this._dpr))
        const c = this.$refs.canvas
        if (c) {
          c.width = Math.max(1, Math.round(this.viewW * this._dpr))
          c.height = Math.max(1, Math.round(this.viewH * this._dpr))
          // element size remains CSS pixels
          c.style.width = '100%'
          c.style.height = '100%'
          // reset and map CSS pixels to backing pixels
          this.ctx = c.getContext('2d')
          this.ctx.setTransform(this._dpr, 0, 0, this._dpr, 0, 0)
        }
      } catch (err) { /* ignore */ }

      // ensure strokes layer matches logical content size and viewport is covered
      this.clampLogicalSize(true)
      this.ensureStrokesLayer()
      this.ensureVisibleCoverage()
      this.updateCursor()
      // initial paint to avoid blank screen until首次交互
      try { this.redraw() } catch(_){}
    },
    ensureStrokesLayer() {
      if (!this.strokesCanvas) {
        this.strokesCanvas = document.createElement('canvas')
        this.strokesCtx = this.strokesCanvas.getContext('2d')
      }
      // scale strokes backing store by DPR so strokes look sharp on HiDPI displays
      const pw = Math.max(1, Math.round(this.width * (this._dpr || 1)))
      const ph = Math.max(1, Math.round(this.height * (this._dpr || 1)))
      if (this.strokesCanvas.width !== pw || this.strokesCanvas.height !== ph) {
        this.strokesCanvas.width = pw
        this.strokesCanvas.height = ph
        // reset context and map CSS/logical pixels to backing pixels
        this.strokesCtx = this.strokesCanvas.getContext('2d')
        try { this.strokesCtx.setTransform(this._dpr || 1, 0, 0, this._dpr || 1, 0, 0) } catch(e){}
        // resizing clears content; mark dirty so we rebuild from strokes
        this._strokesDirty = true
      } else {
        // ensure transform is set even if size unchanged
        try { this.strokesCtx.setTransform(this._dpr || 1, 0, 0, this._dpr || 1, 0, 0) } catch(e){}
      }
    },
    // Ensure the logical board (this.width/height) covers the currently visible viewport
    // after transforms such as zoom/pan/rotate. If not, expand the logical size and adjust
    // pan offsets so visual content doesn't jump.
    ensureVisibleCoverage() {
      try {
        // respect a one-shot skip when snapping to 'see-all' min zoom
        if (this._skipCoverageOnce) { this._skipCoverageOnce = false; return }
        const s = this.displayScale || 1
        // When zoomed out to 'see-all' (contain) or smaller, do not expand the logical canvas
        try {
          const cmin = (typeof this.computeContainRotated === 'function') ? this.computeContainRotated() : this.computeContain()
          // allow small numeric drift; if at-or-below contain within 0.1% margin, skip expansion
          if (s <= cmin * (1 + 1e-3)) return
        } catch(_) {}
        const viewportLogicalW = (this.viewW || 1) / s
        const viewportLogicalH = (this.viewH || 1) / s
        const buffer = this._expandBuffer
        const minOverflow = 1 // logical px; ignore micro overflow to avoid jittery growth
        // Compute viewport rectangle in logical coordinates
        const centerLogicalX = this.width/2 - this.panX
        const centerLogicalY = this.height/2 - this.panY
        const leftLogical = centerLogicalX - viewportLogicalW/2
        const topLogical = centerLogicalY - viewportLogicalH/2
        const rightLogical = leftLogical + viewportLogicalW
        const bottomLogical = topLogical + viewportLogicalH

  let changed = false
        let addLeft = 0, addRight = 0, addTop = 0, addBottom = 0

        if (leftLogical < -minOverflow) {
          addLeft = Math.ceil((-leftLogical) + buffer)
        }
        if (rightLogical > this.width + minOverflow) {
          addRight = Math.ceil((rightLogical - this.width) + buffer)
        }
        if (topLogical < -minOverflow) {
          addTop = Math.ceil((-topLogical) + buffer)
        }
        if (bottomLogical > this.height + minOverflow) {
          addBottom = Math.ceil((bottomLogical - this.height) + buffer)
        }

        // apply caps: do not exceed max logical size; distribute capacity per direction
  const hadHorizontalOverflow = !!(addLeft || addRight)
  if (addLeft || addRight) {
          const remainingW = Math.max(0, this._maxLogicalW - this.width)
          let effLeft = Math.min(addLeft, remainingW)
          let effRight = Math.min(addRight, remainingW - effLeft)
          // if left consumed all capacity but right overflow is larger, try prioritizing larger overflow side
          if ((addRight > addLeft) && (effLeft === addLeft)) {
            // redistribute: fill right first within remainingW
            const cap = Math.max(0, this._maxLogicalW - this.width)
            effRight = Math.min(addRight, cap)
            effLeft = Math.min(addLeft, Math.max(0, cap - effRight))
          }
          if (effLeft || effRight) {
            const oldW = this.width
            this.width = oldW + effLeft + effRight
            // keep opposite edge fixed
            this.panX += (effRight - effLeft) / 2
            changed = true
          }
        }
        const hadVerticalOverflow = !!(addTop || addBottom)
        if (addTop || addBottom) {
          const remainingH = Math.max(0, this._maxLogicalH - this.height)
          let effTop = Math.min(addTop, remainingH)
          let effBottom = Math.min(addBottom, remainingH - effTop)
          if ((addBottom > addTop) && (effTop === addTop)) {
            const cap = Math.max(0, this._maxLogicalH - this.height)
            effBottom = Math.min(addBottom, cap)
            effTop = Math.min(addTop, Math.max(0, cap - effBottom))
          }
          if (effTop || effBottom) {
            const oldH = this.height
            this.height = oldH + effTop + effBottom
            this.panY += (effBottom - effTop) / 2
            changed = true
          }
        }
        if (changed) {
          // ensure strokes layer and redraw to include newly exposed area
          this.ensureStrokesLayer()
          this.redraw()
        } else {
          // 没有扩展成功但存在溢出，说明已达到最大边界
          if (hadHorizontalOverflow || hadVerticalOverflow) {
            this.showHUD('已到达画布最大边界', 1200)
          }
        }
      } catch (err) { /* ignore */ }
    },
    redrawStrokesLayer() {
      this.ensureStrokesLayer()
      const sctx = this.strokesCtx
      // clear in logical coordinates (strokesCtx has DPR transform set)
      try { sctx.clearRect(0,0,this.width,this.height) } catch(e){ sctx.clearRect(0,0,this.strokesCanvas.width,this.strokesCanvas.height) }
      for (const s of this.strokes) {
        this.drawStrokeOn(sctx, s)
      }
      // 将进行中的笔迹也绘制到笔迹层，确保橡皮擦即时效果
      if (this.drawing && this.currentStroke) {
        this.drawStrokeOn(sctx, this.currentStroke)
      }
    },
    // 快速橡皮擦增量绘制：仅在笔迹层上绘制最新一段，避免每帧重建整层
    _fastEraseSegment(stroke, isFirst=false) {
      try {
        if (!stroke || !stroke.eraser) return
        const pts = stroke.points || []
        if (!pts.length) return
        const sctx = this.strokesCtx
        if (!sctx) return
        sctx.save()
        sctx.globalCompositeOperation = 'destination-out'
        sctx.globalAlpha = 1
        sctx.lineJoin = 'round'
        sctx.lineCap = 'round'
        sctx.lineWidth = stroke.size || this.eraserSize || 20
        // 如果只有一个点，盖一个小圆点；否则只连最后两点
        if (pts.length === 1 || isFirst) {
          const p0 = pts[0]
          const r = Math.max(0.5, (stroke.size || this.eraserSize || 20) * 0.5)
          sctx.beginPath(); sctx.arc(p0.x, p0.y, r, 0, Math.PI*2); sctx.closePath(); sctx.fill()
        }
        if (pts.length >= 2) {
          const a = pts[pts.length-2]
          const b = pts[pts.length-1]
          sctx.beginPath(); sctx.moveTo(a.x, a.y); sctx.lineTo(b.x, b.y); sctx.stroke()
        }
      } catch(_) { /* ignore */ } finally { try{ this.strokesCtx && this.strokesCtx.restore() }catch(__){} }
    },
    mapEventToCanvas(ev) {
      const rect = this.$refs.canvas.getBoundingClientRect()
      // screen to canvas local pixels
      let px = (ev.clientX - rect.left)
      let py = (ev.clientY - rect.top)
      // center around canvas center
      px -= this.viewW / 2
      py -= this.viewH / 2
      // inverse overall scale (displayScale)
      const s = this.displayScale || 1
      px /= s; py /= s
      // inverse rotation
      const ang = - (this.rotation * Math.PI / 180)
      const cos = Math.cos(ang), sin = Math.sin(ang)
      const rx = px * cos - py * sin
      const ry = px * sin + py * cos
      // translate back into logical content coords
      const x = rx + this.width / 2 - this.panX
      const y = ry + this.height / 2 - this.panY
      return { x, y }
    },
  onPointerDown(e) {
      // 粘贴预览优先：命中可拖动，点框外视为失焦并确认
      if (this.pasteActive && this.pasteStroke && this.pasteStroke.box) {
        const p = this.mapEventToCanvas(e)
        const b = this.pasteStroke.box
        // 优先命中句柄
        const h = this._hitOverlayHandle(p, this.pasteStroke)
        if (h) {
          this._handleActive = h
          this._handleStart = { p0: p, box0: { ...b }, angle0: this.pasteStroke.angle||0 }
          try{ e.preventDefault() }catch(_){ }
          return
        }
        const hit = (p.x>=b.x && p.x<=b.x+b.w && p.y>=b.y && p.y<=b.y+b.h)
        if (hit) {
          this._pasteDragging = true
          this._pasteDragStart = { x: p.x, y: p.y }
          this._pasteStartBox = { x: b.x, y: b.y, w: b.w, h: b.h }
          try{ e.preventDefault() }catch(_){ }
          return
        } else {
          // 失焦：直接确认落入绘画层
          this._commitPaste()
          // 继续后续流程
        }
      }
      // starting any new interaction cancels pending shape preview
      try { this.cancelShapePreview() } catch(_) {}
      // also cancel hold timers and reset suppression
      this.cancelShapeHold()
      this._suppressNextPointerUp = false
      // If the window just regained focus, swallow the first click to avoid an unintended dot
      try {
        const now = (typeof performance!=='undefined' && performance.now) ? performance.now() : Date.now()
        const focusedRecently = (now - (this._lastFocusAt || 0)) < 350
        if (this._suppressNextClickAfterFocus || focusedRecently) {
          this._suppressNextClickAfterFocus = false
          this._pressStartScreen = null
          this.pressStartPoint = null
          this.pressActive = false
          this.pendingPoint = null
          this._suppressNextPointerUp = true
          return
        }
      } catch(_) {}
      if (e.button === 2) { // right button pressed for RMB+wheel rotate
        this._rmbActive = true
        return
      }
      if (e.button !== undefined && e.button !== 0 && e.button !== 1) return
      e.preventDefault()
      // Pan mode or middle mouse: begin panning instead of drawing
      if (this.panMode || e.button === 1) {
        this.isPanning = true
        this._tempPanByMiddle = (e.button === 1) && !this.panMode
        this._lastPanScreen = { x: e.clientX, y: e.clientY }
        if (this.$refs.canvas) this.$refs.canvas.style.cursor = 'grabbing'
        // cancel any pending draw timer
        this.pressActive = false
        clearTimeout(this.longPressTimer)
        this.pressStartPoint = null
        return
      }
      // 准备开始绘制，但不立即落笔；等待微小位移阈值后再开始，避免获得焦点时出现黑点
      const p = this.mapEventToCanvas(e)
      this.pendingPoint = p
      this.pressStartPoint = p
      this._pressStartScreen = { x: e.clientX, y: e.clientY }
      this.pressActive = true
      clearTimeout(this.longPressTimer)
      try{ this._pressDownAt = (performance && performance.now) ? performance.now() : Date.now() }catch(_){ this._pressDownAt = Date.now() }
    },
    onPointerMove(e) {
      // 粘贴预览：拖动移动
        if (this.pasteActive && this._pasteDragging && this.pasteStroke) {
        const p = this.mapEventToCanvas(e)
        const dx = p.x - (this._pasteDragStart?.x || p.x)
        const dy = p.y - (this._pasteDragStart?.y || p.y)
        const b0 = this._pasteStartBox || this.pasteStroke.box
        const b = this.pasteStroke.box
        b.x = b0.x + dx
        b.y = b0.y + dy
        this.invalidate()
        return
      }
        // 粘贴预览：句柄缩放/旋转
        if (this.pasteActive && this._handleActive && this.pasteStroke) {
          const p = this.mapEventToCanvas(e)
          this._dragOverlayHandle(p)
          this.invalidate()
          return
        }
      if (this.isPanning) {
        if (this._lastPanScreen) {
          const dx = e.clientX - this._lastPanScreen.x
          const dy = e.clientY - this._lastPanScreen.y
          const s = this.displayScale || 1
          const ang = - (this.rotation * Math.PI / 180)
          const cos = Math.cos(ang), sin = Math.sin(ang)
          const wx = (dx * cos - dy * sin) / s
          const wy = (dx * sin + dy * cos) / s
          this.panX += wx
          this.panY += wy
          this.invalidate()
        }
        this._lastPanScreen = { x: e.clientX, y: e.clientY }
        return
      }
      // cancel preview if pointer moves too far while hovering
      if (this._shapePreview && this._previewAnchor) {
        const dx = e.clientX - this._previewAnchor.x
        const dy = e.clientY - this._previewAnchor.y
        if ((dx*dx + dy*dy) > 36) { // >6px movement cancels
          this._previewMoved = true
          this.cancelShapePreview()
        }
      }
      if (!this.drawing) {
        if (this.pressActive) {
          // 忽略获得焦点时的微动；超过阈值后才真正开始绘制
          const sx = e.clientX - (this._pressStartScreen ? this._pressStartScreen.x : e.clientX)
          const sy = e.clientY - (this._pressStartScreen ? this._pressStartScreen.y : e.clientY)
          const dist2 = sx*sx + sy*sy
          const startMoveThreshold = 0 // pixels^2 (~1.4px)
          if (dist2 < startMoveThreshold) return
          // 开始绘制
          const cur = this.mapEventToCanvas(e)
          try { this.commitPendingBrushColor() } catch(_) {}
          this.pushUndoSnapshot()
          this.redoStack = []
          this.drawing = true
          const start = this.pressStartPoint || cur
          this.currentStroke = { points: [start, cur], color: this.erasing ? '#000000' : this.brushColor, size: this.erasing ? this.eraserSize : this.brushSize, eraser: this.erasing, style: this.brushStyle }
          // 取消 pending 状态
          this.pressActive = false
          clearTimeout(this.longPressTimer)
          this._pressStartScreen = null
          // 橡皮擦首段采用增量绘制，避免整层重绘
          if (this.erasing) {
            this.ensureStrokesLayer()
            this._fastEraseSegment(this.currentStroke, true)
            this.invalidate()
          } else {
            this.redraw()
          }
          return
        }
        return
      }
      const p = this.mapEventToCanvas(e)
      this.currentStroke.points.push(p)
      // 橡皮擦在笔迹层做增量擦除 + 轨迹降采样
      if (this.currentStroke && this.currentStroke.eraser) {
        const pts = this.currentStroke.points
        const n = pts.length
        const size = this.currentStroke.size || this.eraserSize || 20
        const minStep = Math.max(1.5, size * 0.25)
        if (n >= 2) {
          const a = pts[n-2], b = pts[n-1]
          const dx = b.x - a.x, dy = b.y - a.y
          if ((dx*dx + dy*dy) < (minStep*minStep)) {
            // 过近：不新增节点，只画一段并覆盖尾点为最新位置
            const sctx = this.strokesCtx
            if (sctx) {
              try {
                sctx.save()
                sctx.globalCompositeOperation = 'destination-out'
                sctx.globalAlpha = 1
                sctx.lineJoin = 'round'
                sctx.lineCap = 'round'
                sctx.lineWidth = size
                sctx.beginPath(); sctx.moveTo(a.x, a.y); sctx.lineTo(b.x, b.y); sctx.stroke()
              } finally { try{ this.strokesCtx && this.strokesCtx.restore() }catch(__){} }
            }
            // 覆盖尾点，保持点数不增长
            pts[n-2] = b
            pts.pop()
          } else {
            // 正常增量段
            this.ensureStrokesLayer()
            this._fastEraseSegment(this.currentStroke)
          }
        } else {
          // 仅1个点：补一个小点
          this.ensureStrokesLayer()
          this._fastEraseSegment(this.currentStroke, true)
        }
      }
      // 合帧重绘以降低指针移动时的卡顿
      this.invalidate()
      // schedule hold-to-preview/commit while still pressing
      if (this.shapeSnapEnabled && !this.erasing && (e.buttons & 1)) {
        this.scheduleShapeHold(e)
      }
    },
    onPointerLeave() { this.cancelShapePreview() },
    onPointerUp(e) {
      if (this._pasteDragging) { this._pasteDragging = false; this._pasteDragStart = null; this._pasteStartBox = null; return }
      if (this._handleActive) { this._handleActive = null; this._handleStart = null; return }
      if (e.button === 2) {
        this._rmbActive = false
        return
      }
      // cancel hold timers on release
      this.cancelShapeHold()
      if (this._suppressNextPointerUp) { this._suppressNextPointerUp = false; return }
      if (this.isPanning) {
        this.isPanning = false
        this._lastPanWorld = null
        this._lastPanScreen = null
        this._tempPanByMiddle = false
        this.updateCursor()
        return
      }
      // end any pending press state
      this.pressActive = false
      clearTimeout(this.longPressTimer)
      this.pressStartPoint = null
      this._pressStartScreen = null
      if (!this.drawing) {
        // treat as a dot click if no movement and not right after window focus
        const now = (typeof performance!=='undefined' && performance.now) ? performance.now() : Date.now()
        const focusedRecently = (now - (this._lastFocusAt || 0)) < 200
        const sx = e.clientX - (this._pressStartScreen ? this._pressStartScreen.x : e.clientX)
        const sy = e.clientY - (this._pressStartScreen ? this._pressStartScreen.y : e.clientY)
        const dist2 = sx*sx + sy*sy
        const startMoveThreshold = 9 // pixels^2 (~3px)
        if (!focusedRecently && this.pressStartPoint && dist2 < startMoveThreshold) {
          // single click -> draw a dot
          const p = this.pressStartPoint
          const dot = { points: [p], color: this.erasing ? '#000000' : this.brushColor, size: this.erasing ? this.eraserSize : this.brushSize, eraser: this.erasing, style: this.brushStyle, dot: true }
          // 记录撤销快照，确保撤销只移除这一笔，不影响之前的粘贴
          this.pushUndoSnapshot(); this.redoStack = []
          this.strokes.push(dot)
          // 非橡皮擦：需要重建笔迹层；橡皮擦：增量已生效，无需重建
          if (!dot.eraser) this._strokesDirty = true
          this.saveAutosave()
          this.redraw()
        }
        this.currentStroke = null
        return
      }
      this.drawing = false
      // discard clicks without movement (avoid empty strokes)
      if (this.currentStroke && this.currentStroke.points.length >= 2) {
        this.strokes.push(this.currentStroke)
        // 非橡皮擦：重建；橡皮擦：已增量绘制，无需重建
        if (!this.currentStroke.eraser) this._strokesDirty = true
        // do not auto-beautify on release; the new gesture handles beautification while holding.
        this.currentStroke = null
        this.saveAutosave()
        this.redraw()
      } else {
        // no visible segment; just clear state and redraw to remove preview
        this.currentStroke = null
        this.redraw()
      }
    },
    // draw a stroke on a specific 2D context (shares logic for screen and export)
    drawStrokeOn(ctx, stroke) {
      if (!stroke) return
      // image stroke: draw an embedded bitmap with placement box
      if (stroke.type === 'image' && stroke.box && stroke.src) {
        if (stroke._importMissing) {
          const b = stroke.box
          const drawMissingPlaceholder = ()=>{
            ctx.save()
            if (typeof stroke.opacity === 'number') ctx.globalAlpha = Math.max(0.3, stroke.opacity)
            ctx.fillStyle = 'rgba(255, 244, 229, 0.95)'
            ctx.strokeStyle = 'rgba(217, 119, 6, 0.9)'
            ctx.lineWidth = Math.max(2, Math.min(b.w, b.h) * 0.015)
            if (ctx.setLineDash) ctx.setLineDash([10, 8])
            ctx.fillRect(-b.w/2, -b.h/2, b.w, b.h)
            ctx.strokeRect(-b.w/2, -b.h/2, b.w, b.h)
            if (ctx.setLineDash) ctx.setLineDash([])
            ctx.fillStyle = '#92400e'
            const titleSize = Math.max(14, Math.min(28, Math.round(Math.min(b.w, b.h) * 0.12)))
            const descSize = Math.max(11, Math.min(18, Math.round(titleSize * 0.62)))
            ctx.textAlign = 'center'
            ctx.textBaseline = 'middle'
            ctx.font = `600 ${titleSize}px sans-serif`
            ctx.fillText('旧版图片缺失', 0, -Math.min(12, titleSize * 0.4))
            ctx.font = `400 ${descSize}px sans-serif`
            ctx.fillText(stroke._missingReason || '导出时未内嵌资源', 0, Math.min(18, descSize * 1.4))
            ctx.restore()
          }
          ctx.save()
          if (stroke.angle) {
            const cx = b.x + b.w/2
            const cy = b.y + b.h/2
            ctx.translate(cx, cy)
            ctx.rotate((stroke.angle||0) * Math.PI / 180)
            drawMissingPlaceholder()
          } else {
            ctx.translate(b.x + b.w/2, b.y + b.h/2)
            drawMissingPlaceholder()
          }
          ctx.restore()
          return
        }
        let img = stroke._img
        if (!img) {
          img = new Image()
          img.decoding = 'sync'
          img.src = stroke.src
          stroke._img = img
          if (!img.complete) {
            img.onload = () => { this._strokesDirty = true; this.redraw && this.redraw() }
            img.onerror = () => { console.warn('image stroke load failed') }
          }
        }
        if (img && img.complete && img.naturalWidth > 0) {
          ctx.save()
          if (typeof stroke.opacity === 'number') ctx.globalAlpha = stroke.opacity
          try { ctx.imageSmoothingEnabled = true; ctx.imageSmoothingQuality = 'high' } catch(_){}
          const b = stroke.box
          if (stroke.angle) {
            const cx = b.x + b.w/2
            const cy = b.y + b.h/2
            ctx.translate(cx, cy)
            ctx.rotate((stroke.angle||0) * Math.PI / 180)
            ctx.drawImage(img, 0, 0, img.width || img.naturalWidth, img.height || img.naturalHeight, -b.w/2, -b.h/2, b.w, b.h)
          } else {
            ctx.drawImage(img, 0, 0, img.width || img.naturalWidth, img.height || img.naturalHeight, b.x, b.y, b.w, b.h)
          }
          ctx.restore()
        }
        return
      }
      // text stroke
      if (stroke.type === 'text' && stroke.box && typeof stroke.text === 'string') {
        const b = stroke.box
        const lines = (stroke.text||'').replace(/\r\n/g, '\n').split('\n')
        const base = Math.max(8, Math.round(stroke.baseSize || 24))
        const lhMult = Math.max(0.6, Number(stroke.lineHeightMult||1.2))
        const lineH = Math.round(base * lhMult)
        // measure natural width using base font
        const off = document.createElement('canvas'); const m = off.getContext('2d')
        const fontStr = `${stroke.fontWeight||'normal'} ${base}px ${stroke.fontFamily||'sans-serif'}`
        m.font = fontStr
        let natW = 0
        const lineWidths = []
        const ls = Number(stroke.letterSpacing||0)
        for (const ln of lines) {
          let wSum = 0
          if (ln.length<=1) {
            wSum = Math.ceil(m.measureText(ln).width || 0)
          } else {
            for (let ci=0; ci<ln.length; ci++) {
              const ch = ln[ci]
              wSum += Math.ceil(m.measureText(ch).width || 0)
              if (ci < ln.length-1) wSum += ls
            }
          }
          lineWidths.push(wSum)
          natW = Math.max(natW, wSum)
        }
        const natH = Math.max(lineH, lines.length * lineH)
        // 填满框：允许放大，最大化贴合；默认：仅在过大时按比例缩小
        let s = 1
        if (natW>0 && natH>0) {
          s = Math.min(b.w / natW, b.h / natH)
          if (!isFinite(s) || s<=0) s = 1
          if (!stroke.fillBox) s = Math.min(1, s) // shrink-only when not fillBox
        }
        ctx.save()
        if (typeof stroke.opacity === 'number') ctx.globalAlpha = stroke.opacity
        ctx.fillStyle = stroke.color || '#000'
        ctx.textBaseline = 'top'
        const drawBlock = ()=>{
          ctx.save()
          ctx.scale(s, s)
          ctx.font = fontStr
          // 背景块
          if (stroke.bgColor && stroke.bgColor !== '#00000000' && stroke.bgColor !== 'transparent') {
            ctx.save(); ctx.fillStyle = stroke.bgColor; ctx.fillRect(0, 0, natW, natH); ctx.restore()
          }
          // 对齐
          const align = (stroke.align||'left')
          // 描边宽转换到缩放前坐标
          const lw = Math.max(0, Number(stroke.textStrokeWidth||0))
          const lwScaled = lw / Math.max(1e-6, s)
          // 阴影（在缩放空间内，值按自然像素使用）
          const shCol = stroke.textShadowColor || 'transparent'
          const shBlur = Math.max(0, Number(stroke.textShadowBlur||0))
          const shX = Number(stroke.textShadowOffsetX||0)
          const shY = Number(stroke.textShadowOffsetY||0)
          if (shCol && shCol !== 'transparent' && (shBlur>0 || shX!==0 || shY!==0)) {
            ctx.shadowColor = shCol
            ctx.shadowBlur = shBlur
            ctx.shadowOffsetX = shX
            ctx.shadowOffsetY = shY
          } else {
            ctx.shadowColor = 'transparent'
            ctx.shadowBlur = 0
            ctx.shadowOffsetX = 0
            ctx.shadowOffsetY = 0
          }
          // 逐字符绘制以支持字间距
          for (let i=0;i<lines.length;i++){
            const ln = lines[i]
            const wLine = lineWidths[i] || 0
            let x = 0
            if (align==='center') x = (natW - wLine)/2
            else if (align==='right') x = (natW - wLine)
            const y = i*lineH
            if (ln.length<=1) {
              // 单字符优化
              if (lwScaled > 0) { ctx.save(); ctx.lineJoin='round'; ctx.miterLimit=2; ctx.strokeStyle = stroke.textStrokeColor || '#000'; ctx.lineWidth = lwScaled; ctx.strokeText(ln, x, y); ctx.restore() }
              ctx.fillText(ln, x, y)
            } else {
              for (let ci=0; ci<ln.length; ci++){
                const ch = ln[ci]
                const wch = Math.ceil(m.measureText(ch).width || 0)
                if (lwScaled > 0) { ctx.save(); ctx.lineJoin='round'; ctx.miterLimit=2; ctx.strokeStyle = stroke.textStrokeColor || '#000'; ctx.lineWidth = lwScaled; ctx.strokeText(ch, x, y); ctx.restore() }
                ctx.fillText(ch, x, y)
                x += wch + (ci<ln.length-1 ? ls : 0)
              }
            }
          }
          ctx.restore()
        }
        if (stroke.angle) {
          const cx = b.x + b.w/2
          const cy = b.y + b.h/2
          ctx.translate(cx, cy)
          ctx.rotate((stroke.angle||0) * Math.PI/180)
          ctx.translate(-b.w/2, -b.h/2)
          drawBlock()
        } else {
          ctx.translate(b.x, b.y)
          drawBlock()
        }
        ctx.restore()
        return
      }
      // handle geometric shapes first (shaped strokes may not have points)
      if (stroke.shape && stroke.shape.type) {
        return this.drawShapeOn(ctx, stroke)
      }
      if (!stroke.points || !stroke.points.length) return
      ctx.save()
      ctx.lineJoin = 'round'
      ctx.lineCap = 'round'
      // brush styles
      if (stroke.eraser) {
        ctx.globalCompositeOperation = 'destination-out'
        ctx.globalAlpha = 1
        ctx.lineWidth = stroke.size
      } else if (stroke.style === 'marker') {
        ctx.globalCompositeOperation = 'multiply'
        ctx.globalAlpha = 0.6
        ctx.lineWidth = stroke.size * 1.2
        ctx.lineCap = 'butt'
        ctx.lineJoin = 'round'
      } else if (stroke.style === 'pencil') {
        ctx.globalCompositeOperation = 'source-over'
        ctx.globalAlpha = 0.85
        ctx.lineWidth = Math.max(1, stroke.size * 0.8)
      } else { // pen default
        ctx.globalCompositeOperation = 'source-over'
        ctx.globalAlpha = 1
        ctx.lineWidth = stroke.size
      }
      ctx.strokeStyle = stroke.color
      const pts = stroke.points
      // special case: single-point stroke -> draw a round dot
      if (pts.length === 1) {
        const p0 = pts[0]
        const r = Math.max(0.5, (stroke.size || 2) * 0.5)
        const prevOp = ctx.globalCompositeOperation
        const prevAlpha = ctx.globalAlpha
        if (stroke.eraser) {
          ctx.globalCompositeOperation = 'destination-out'
          ctx.globalAlpha = 1
        } else {
          ctx.fillStyle = stroke.color
        }
        ctx.beginPath(); ctx.arc(p0.x, p0.y, r, 0, Math.PI * 2); ctx.closePath(); ctx.fill()
        ctx.globalCompositeOperation = prevOp
        ctx.globalAlpha = prevAlpha
        ctx.restore()
        return
      }
      // quadratic smoothing between points
      ctx.beginPath()
      ctx.moveTo(pts[0].x, pts[0].y)
      if (pts.length === 2) {
        ctx.lineTo(pts[1].x, pts[1].y)
      } else {
        for (let i = 1; i < pts.length - 1; i++) {
          const midX = (pts[i].x + pts[i + 1].x) / 2
          const midY = (pts[i].y + pts[i + 1].y) / 2
          ctx.quadraticCurveTo(pts[i].x, pts[i].y, midX, midY)
        }
        const last = pts[pts.length - 1]
        ctx.lineTo(last.x, last.y)
      }
      ctx.stroke()
      // extra pass for pencil to add subtle texture
      if (!stroke.eraser && stroke.style === 'pencil') {
        ctx.globalAlpha = 0.28
        const w = Math.max(0.5, (stroke.size * 0.45))
        ctx.lineWidth = w
        ctx.beginPath()
        ctx.moveTo(pts[0].x, pts[0].y)
        if (pts.length === 2) {
          ctx.lineTo(pts[1].x, pts[1].y)
        } else {
          for (let i = 1; i < pts.length - 1; i++) {
            const midX = (pts[i].x + pts[i + 1].x) / 2
            const midY = (pts[i].y + pts[i + 1].y) / 2
            ctx.quadraticCurveTo(pts[i].x, pts[i].y, midX, midY)
          }
          const last = pts[pts.length - 1]
          ctx.lineTo(last.x, last.y)
        }
        ctx.stroke()
      }
      ctx.restore()
    },
    // draw beautified geometric shape
    drawShapeOn(ctx, stroke) {
      const shape = stroke.shape
      if (!shape) return
      ctx.save()
      // line style like in normal brush
      // prefer rounded joins to avoid sharp spikes at polygon corners
      ctx.lineJoin = 'round'
      ctx.miterLimit = 2
      ctx.lineCap = 'round'
      if (stroke.eraser) {
        ctx.globalCompositeOperation = 'destination-out'
        ctx.globalAlpha = 1
        ctx.lineWidth = stroke.size
      } else if (stroke.style === 'marker') {
        ctx.globalCompositeOperation = 'multiply'
        ctx.globalAlpha = 0.6
        ctx.lineWidth = stroke.size * 1.2
        ctx.lineCap = 'butt'
        ctx.lineJoin = 'round'
      } else if (stroke.style === 'pencil') {
        ctx.globalCompositeOperation = 'source-over'
        ctx.globalAlpha = 0.95
        ctx.lineWidth = Math.max(1, stroke.size * 0.9)
      } else {
        ctx.globalCompositeOperation = 'source-over'
        ctx.globalAlpha = 1
        ctx.lineWidth = stroke.size
      }
  ctx.strokeStyle = stroke.color || '#000'
  if (stroke.preview) { try { ctx.setLineDash([8,6]) } catch(_){} }
      const t = shape.type
      if (t === 'line') {
        const { p1, p2 } = shape
        ctx.beginPath(); ctx.moveTo(p1.x, p1.y); ctx.lineTo(p2.x, p2.y); ctx.stroke()
      } else if (t === 'circle') {
        const { c, r } = shape
        ctx.beginPath(); ctx.arc(c.x, c.y, Math.max(0,r), 0, Math.PI*2); ctx.stroke()
        // soften seam where full-circle stroke starts/ends
        this._softenSeam(ctx, c.x + r, c.y, stroke)
      } else if (t === 'arc') {
        const { c, r, a0, a1, ccw } = shape
        ctx.beginPath(); ctx.arc(c.x, c.y, Math.max(0,r), a0, a1, !!ccw); ctx.stroke()
      } else if (t === 'rect') {
        const { p } = shape // p: array of 4 corners in order
        ctx.beginPath(); ctx.moveTo(p[0].x, p[0].y)
        for (let i=1;i<p.length;i++) ctx.lineTo(p[i].x, p[i].y)
        ctx.closePath(); ctx.stroke()
        // blur the closure joint slightly
        this._softenSeam(ctx, p[0].x, p[0].y, stroke)
      } else if (t === 'star') {
        const { c, R, r, rot } = shape
        ctx.beginPath();
        const pts = this._starPoints(c, R, r || (R*0.38196601125), rot || 0)
        ctx.moveTo(pts[0].x, pts[0].y)
        for(let i=1;i<pts.length;i++) ctx.lineTo(pts[i].x, pts[i].y)
        ctx.closePath(); ctx.stroke()
        this._softenSeam(ctx, pts[0].x, pts[0].y, stroke)
      } else if (t === 'polygon') {
        const { p } = shape // p: array of vertices
        if (p && p.length>=2) {
          ctx.beginPath(); ctx.moveTo(p[0].x, p[0].y)
          for (let i=1;i<p.length;i++) ctx.lineTo(p[i].x, p[i].y)
          if (shape.closed !== false) ctx.closePath()
          ctx.stroke()
          if (shape.closed !== false) this._softenSeam(ctx, p[0].x, p[0].y, stroke)
        }
      } else {
        // fallback to polyline from points if unknown
        const pts = stroke.points
        ctx.beginPath(); ctx.moveTo(pts[0].x, pts[0].y)
        for (let i=1;i<pts.length;i++) ctx.lineTo(pts[i].x, pts[i].y)
        ctx.stroke()
      }
      ctx.restore()
    },
    _softenSeam(ctx, x, y, stroke){
      // draw a tiny blurred dot at the start/closure to hide sharp seam
      try{
        ctx.save()
        const blurPx = Math.max(0, this.dbg?.seamBlurPx ?? 0.6)
        ctx.filter = `blur(${blurPx}px)`
        ctx.globalAlpha = Math.max(0, Math.min(1, this.dbg?.seamBlurAlpha ?? 0.5))
        ctx.fillStyle = stroke.color || '#000'
        const rr = Math.max(0.5, (stroke.size||2) * (this.dbg?.seamDotScale ?? 0.55))
        ctx.beginPath(); ctx.arc(x, y, rr, 0, Math.PI*2); ctx.fill()
      } finally { try{ ctx.restore() }catch(_){} }
    },
    _starPoints(c, R, r, rot){
      const pts=[]; const cx=c.x, cy=c.y; const base = (rot||0) - Math.PI/2
      for(let i=0;i<10;i++){
        const ang = base + i*(Math.PI/5)
        const rad = (i%2===0)? R : r
        pts.push({ x: cx + Math.cos(ang)*rad, y: cy + Math.sin(ang)*rad })
      }
      return pts
    },
    redraw() {
      const ctx = this.ctx
      if (!ctx) return
      // ensure transform maps CSS pixels to backing pixels (DPR)
      try { ctx.setTransform(this._dpr, 0, 0, this._dpr, 0, 0) } catch(e){}
      // clear and fill entire viewport canvas to avoid exposing site background
      ctx.clearRect(0,0,this.viewW,this.viewH)
      ctx.fillStyle = this.bgColor
      ctx.fillRect(0,0,this.viewW,this.viewH)

  // apply view transform: center -> rotate -> scale -> translate to logical content origin (with pan)
      ctx.save()
      ctx.translate(this.viewW/2, this.viewH/2)
      ctx.rotate(this.rotation * Math.PI / 180)
      ctx.scale(this.displayScale, this.displayScale)
  ctx.translate(-this.width/2 + this.panX, -this.height/2 + this.panY)

      // draw texture grid if needed (rotates with content) and extend beyond logical bounds
      if (this.texture === 'grid') {
        ctx.save(); ctx.strokeStyle = 'rgba(0,0,0,0.06)'; ctx.lineWidth = 1
        const step = 24
        const padX = this.width * 2
        const padY = this.height * 2
        for (let x=-padX; x<=this.width+padX; x+=step) { ctx.beginPath(); ctx.moveTo(x, -padY); ctx.lineTo(x, this.height+padY); ctx.stroke() }
        for (let y=-padY; y<=this.height+padY; y+=step) { ctx.beginPath(); ctx.moveTo(-padX, y); ctx.lineTo(this.width+padX, y); ctx.stroke() }
        ctx.restore()
      }
      // draw gray overlay for areas outside logical content; works under rotation using even-odd fill
      try {
        ctx.save()
        ctx.fillStyle = 'rgba(200,200,200,0.36)'
        // big margin in logical units to ensure outer rect fully covers the visible viewport under any rotation
        const s = this.displayScale || 1
        const margin = Math.max(this.width, this.height) + Math.max(this.viewW, this.viewH) / s * 2
        ctx.beginPath()
        // outer huge rect
        ctx.rect(-margin, -margin, this.width + margin * 2, this.height + margin * 2)
        // inner content rect (hole)
        ctx.rect(0, 0, this.width, this.height)
        ctx.fill('evenodd')
        ctx.restore()
      } catch(e) { /* ignore gray overlay errors */ }

      // 绘制数字拼图叠加（若存在），坐标为逻辑坐标
      try {
        if (this.puzzleOverlay && this.puzzleOverlayBox) {
          const b = this.puzzleOverlayBox
          // 开启平滑缩放，降低锯齿
          ctx.imageSmoothingEnabled = true
          try{ ctx.imageSmoothingQuality = 'high' }catch(_){ }
          ctx.drawImage(this.puzzleOverlay, 0, 0, this.puzzleOverlay.width, this.puzzleOverlay.height, b.x, b.y, b.w, b.h)
        }
      } catch(_){}
      // 仅在结构性变更时重建笔迹图层（尺寸变化、撤销/添加、图片笔迹加载等）。
      // 橡皮擦已走增量绘制，无需每帧重建整层。
      if (this._strokesDirty) {
        this.redrawStrokesLayer()
        this._strokesDirty = false
      }
      // draw strokesCanvas (source in backing pixels) into logical content area (dest in logical units)
      try {
        ctx.drawImage(this.strokesCanvas, 0, 0, this.strokesCanvas.width, this.strokesCanvas.height, 0, 0, this.width, this.height)
      } catch (e) {
        // fallback: direct draw
        try { ctx.drawImage(this.strokesCanvas, 0, 0) } catch(_){ }
      }

      // 正在绘制且不是橡皮擦时，直接在主画布上叠加当前笔迹，避免每帧重建离屏层
      try {
        if (this.drawing && this.currentStroke && !this.currentStroke.eraser) {
          this.drawStrokeOn(ctx, this.currentStroke)
        }
      } catch(_){}

      // 粘贴预览层：始终绘制在最上方，带虚线框
      try {
        if (this.pasteActive && this.pasteStroke) {
          this.drawStrokeOn(ctx, this.pasteStroke)
          // 绘制虚线边框（忽略旋转时的外接框）
          const b = this.pasteStroke.box
          ctx.save()
          ctx.setLineDash && ctx.setLineDash([6,6])
          ctx.globalAlpha = 0.9
          ctx.strokeStyle = '#3B82F6'
          ctx.lineWidth = 1
          ctx.strokeRect(b.x, b.y, b.w, b.h)
          // 绘制句柄（角/边 + 旋转手柄）
          const s = Math.max(0.5, this.displayScale || 1)
          const hs = Math.max(6, Math.round(8 / s)) // 统一屏幕大小
          const r = Math.max(6, Math.round(10 / s))
          const handle = (x,y)=>{ ctx.fillStyle='#3B82F6'; ctx.fillRect(x-hs/2, y-hs/2, hs, hs); ctx.strokeStyle='#ffffff'; ctx.lineWidth=1; ctx.strokeRect(x-hs/2+0.5, y-hs/2+0.5, hs-1, hs-1) }
          // corners
          handle(b.x, b.y); handle(b.x+b.w, b.y); handle(b.x, b.y+b.h); handle(b.x+b.w, b.y+b.h)
          // edges mid
          handle(b.x+b.w/2, b.y); handle(b.x+b.w/2, b.y+b.h); handle(b.x, b.y+b.h/2); handle(b.x+b.w, b.y+b.h/2)
          // rotation handle (top center, connected line)
          const off = Math.max(18, Math.round(28 / s))
          const rx = b.x + b.w/2, ry = b.y - off
          ctx.beginPath(); ctx.moveTo(b.x + b.w/2, b.y); ctx.lineTo(rx, ry); ctx.strokeStyle='#3B82F6'; ctx.lineWidth=1; ctx.stroke()
          ctx.beginPath(); ctx.arc(rx, ry, r/2, 0, Math.PI*2); ctx.fillStyle='#3B82F6'; ctx.fill(); ctx.strokeStyle='#ffffff'; ctx.lineWidth=1; ctx.stroke()
          ctx.restore()
        }
      } catch(_){ }

      // preview overlay for shape beautification (dashed)
      try {
        if (this._shapePreview && this._shapePreview.shape) {
          const prev = this._shapePreview
          const tmp = { color: prev.color, size: prev.size, eraser: false, style: prev.style, shape: prev.shape, preview: true }
          // dashed overlay
          ctx.save(); ctx.setLineDash([8,6]); ctx.globalAlpha = 0.9; ctx.restore()
          this.drawShapeOn(ctx, tmp)
        }
      } catch(_) {}
      ctx.restore()
    },
    zoomIn() {
      const current = this.displayScale || 1
      const minTarget = Math.max(this.minDisplayScale, this.computeContain())
      const target = Math.max(minTarget, Math.min(this.maxDisplayScale, current + 0.1))
      // set scale to achieve target display scale
      this.scale = target / (this.computeFit() || 1)
      // If we are snapping against the minTarget equal to contain-fit, skip expansion once and recentre
      if (Math.abs(target - minTarget) < 1e-6 && Math.abs(minTarget - this.computeContain()) < 1e-6) {
        this._skipCoverageOnce = true
        this.panX = 0; this.panY = 0
      }
      // after potential coverage changes, fit may change; reapply to keep target visually stable
      this.ensureVisibleCoverage()
      this.scale = target / (this.computeFit() || 1)
      this.redraw(); this.updateCursor(); this.updateToolbarPositions()
    },
    zoomOut() {
      const current = this.displayScale || 1
      const minTarget = Math.max(this.minDisplayScale, this.computeContain())
      const target = Math.max(minTarget, Math.min(this.maxDisplayScale, current - 0.1))
      this.scale = target / (this.computeFit() || 1)
      // If we hit the minimum bound (contain-fit), skip expansion once and recenter
      if (Math.abs(target - minTarget) < 1e-6 && Math.abs(minTarget - this.computeContain()) < 1e-6) {
        this._skipCoverageOnce = true
        this.panX = 0; this.panY = 0
      }
      this.ensureVisibleCoverage()
      this.scale = target / (this.computeFit() || 1)
      this.redraw(); this.updateCursor(); this.updateToolbarPositions()
    },
    // Fit the entire (possibly rotated) logical canvas into the viewport
    fitToView() {
      const contain = this.computeContainRotated()
      const target = Math.max(this.minDisplayScale, Math.min(this.maxDisplayScale, contain))
      // set target scale via user scale = target / fit (fit uses cover logic, stable per our pipeline)
      this._skipCoverageOnce = true
      this.panX = 0; this.panY = 0
      this.scale = target / (this.computeFit() || 1)
      this.ensureVisibleCoverage()
      this.scale = target / (this.computeFit() || 1)
      this.redraw(); this.updateCursor(); this.updateToolbarPositions();
      // HUD hint
      try { this.showHUD('已看全画布'); } catch(_) {}
    },
  rotate90(dir='cw') { this.rotation = (this.rotation + (dir==='cw'?90:-90) + 360) % 360; this.updateToolbarPositions(); this.redraw() },
    onTextureChange() { this.redraw() },
    onBgColorChange() { this.redraw() },
  toggleEraser() { this.erasing = !this.erasing; this.panMode = false; this.updateCursor() },

    // switch brush with icon click
  selectBrush(name) { this.brushStyle = name; this.erasing = (name==='eraser'); this.panMode = false; this.updateCursor() },

    // toolbar attachment: compute positions of toolbars relative to visible canvas
    updateToolbarPositions() {
      // toolbars are fixed to the browser edges so they remain visible even when canvas is rotated
      // left: vertical center on left edge; right: vertical center on right edge; bottom: centered at bottom
      this.leftStyle = { position: 'fixed', left: '12px', top: '50%', transform: 'translateY(-50%)', zIndex: 60 }
      this.rightStyle = { position: 'fixed', right: '12px', top: '50%', transform: 'translateY(-50%)', zIndex: 60 }
      this.bottomStyle = { position: 'fixed', left: '50%', transform: 'translateX(-50%)', bottom: '12px', zIndex: 60, width: 'auto', maxWidth: '90%' }
    },

    // display fullscreen (requestFullscreen) toggling
    toggleDisplayFullscreen() {
      try {
        if (!this.isDisplayFullscreen) {
          const el = this.$el
          if (el.requestFullscreen) el.requestFullscreen()
          else if (el.webkitRequestFullscreen) el.webkitRequestFullscreen()
        } else {
          if (document.exitFullscreen) document.exitFullscreen()
          else if (document.webkitExitFullscreen) document.webkitExitFullscreen()
        }
      } catch (e) { console.warn('fullscreen failed', e) }
    },
    onFullscreenChange() {
      this.isDisplayFullscreen = !!document.fullscreenElement
      // when exiting display fullscreen via ESC, keep webpage in web-full mode (we already hide chrome via route)
      this.updateToolbarPositions()
      // force a resize+redraw to avoid black screen until next interaction
      this.resizeCanvas()
      clearTimeout(this._fsTimer)
      this._fsTimer = setTimeout(()=>{ this.resizeCanvas() }, 50)
    },

    // generate and set a custom cursor for the canvas based on brush size/style
    updateCursor() {
      try {
        if (this.isPanning || this.panMode) {
          if (this.$refs.canvas) this.$refs.canvas.style.cursor = this.isPanning ? 'grabbing' : 'grab'
          return
        }
        const curData = this.generateCursorDataUrl()
        const hx = Math.max(0, Math.min(255, Math.round(curData.hx || 0)))
        const hy = Math.max(0, Math.min(255, Math.round(curData.hy || 0)))
        const cur = `url(${curData.url}) ${hx} ${hy}, crosshair`
        if (this.$refs.canvas) this.$refs.canvas.style.cursor = cur
        // keep cursor change local to canvas only (do not change entire document)
      } catch (e) { console.error('updateCursor', e) }
    },
    generateCursorDataUrl() {
      // 根据当前笔刷大小和显示缩放生成更醒目的光标
    const effectiveSize = this.erasing ? this.eraserSize : this.brushSize
  // ringEffective controls the visual ring size shown in the cursor.
  // Previously we capped eraser growth to avoid massive rings; remove that cap so the circle
  // continues to reflect eraserSize changes. Bitmap size is still limited later by MAX_CURSOR_PIX.
  const ringEffective = effectiveSize
    const displayScaleSafe = (this.displayScale || 1)
    // visual value (px) used to compute cursor canvas size (based on ringEffective to cap eraser growth)
    const visualForCursor = Math.max(6, ringEffective * displayScaleSafe)
    // limit cursor bitmap size to a browser-friendly maximum. increase cap so the
    // existing cursor's ring can grow larger before hitting browser limits.
    const MAX_CURSOR_PIX = 512
    const rawSize = Math.round(visualForCursor * 4)
    const size = Math.max(40, Math.min(MAX_CURSOR_PIX, rawSize))
    const c = document.createElement('canvas'); c.width = size; c.height = size
    const ctx = c.getContext('2d')
    ctx.clearRect(0,0,size,size)
    // 以 nib（笔尖/作用中心）为圆心
    const cx = size/2, cy = size/2
    // compute a radius inside the cursor bitmap that corresponds to half the stroke width
    const scale = rawSize > 0 ? (size / rawSize) : 1
    const radius = Math.max(4, Math.round((visualForCursor / 2) * scale))

      // 高对比环与十字
      const hi = this.contrastingColor(this.bgColor)
      const lo = hi === '#F9FAFB' ? 'rgba(17,24,39,0.85)' : 'rgba(249,250,251,0.95)'
      // draw ring/indicator
      // dashed/dotted ring to reduce visual weight
      const dashLen = Math.max(2, Math.round(radius / 3))
      const gapLen = dashLen
      if (this.erasing) {
        // very faint filled area
        ctx.save()
        ctx.globalAlpha = 0.08
        ctx.fillStyle = hi
        ctx.beginPath(); ctx.arc(cx, cy, radius, 0, Math.PI*2); ctx.fill()
        ctx.restore()
        // dashed ring (slightly more transparent)
        ctx.save()
        ctx.setLineDash([dashLen, gapLen])
        ctx.lineCap = 'round'
        ctx.globalAlpha = 0.10
        ctx.strokeStyle = lo
        ctx.lineWidth = Math.max(2, Math.round(radius * 0.14))
        ctx.beginPath(); ctx.arc(cx, cy, radius, 0, Math.PI*2); ctx.stroke()
        ctx.restore()
        // small crosshair
        ctx.save(); ctx.globalAlpha = 0.9; ctx.strokeStyle = hi; ctx.lineWidth = 1
        ctx.beginPath(); ctx.moveTo(cx-6, cy); ctx.lineTo(cx+6, cy); ctx.stroke()
        ctx.beginPath(); ctx.moveTo(cx, cy-6); ctx.lineTo(cx, cy+6); ctx.stroke()
        ctx.restore()
      } else {
        // brush mode: very faint dashed ring plus a subtle crosshair
        ctx.save()
        ctx.setLineDash([dashLen, gapLen])
        ctx.lineCap = 'round'
        ctx.globalAlpha = 0.06
        ctx.strokeStyle = lo
        ctx.lineWidth = Math.max(1, Math.round(radius * 0.12))
        ctx.beginPath(); ctx.arc(cx, cy, radius, 0, Math.PI*2); ctx.stroke()
        ctx.restore()
        // small crosshair for precision
        ctx.save(); ctx.globalAlpha = 0.9; ctx.strokeStyle = hi; ctx.lineWidth = 1
        ctx.beginPath(); ctx.moveTo(cx-5, cy); ctx.lineTo(cx+5, cy); ctx.stroke()
        ctx.beginPath(); ctx.moveTo(cx, cy-5); ctx.lineTo(cx, cy+5); ctx.stroke()
        ctx.restore()
      }
      // 仅在橡皮擦模式下保留中心点提示，画笔模式不绘制中心小圆点
      if (this.erasing) {
        ctx.save()
        ctx.beginPath(); ctx.arc(cx, cy, Math.max(1, Math.round(radius*0.22)), 0, Math.PI*2)
        ctx.fillStyle = lo
        ctx.globalAlpha = 0.35
        ctx.fill()
        ctx.restore()
      }

  // 在 nib 位置绘制工具图标，并让 nib 作为热点
  // 再调大一点图标，并让笔尖朝左下，笔身在右上（顺时针 135°）
  const angleDL = Math.PI * 3/4 // 135°，顺时针
  const thickness = Math.max(4, Math.round(radius * 0.5))
  const bodyLen = Math.max(12, Math.round(radius * 1.6))

      const drawPenIcon = ()=>{
        ctx.save(); ctx.translate(cx, cy); ctx.rotate(angleDL)
        // 笔尖三角（尖端在 0,0）
        ctx.fillStyle = '#d4d4d4'; ctx.beginPath(); ctx.moveTo(0, 0); ctx.lineTo(-Math.min(6, thickness*1.8), thickness/2); ctx.lineTo(-Math.min(6, thickness*1.8), -thickness/2); ctx.closePath(); ctx.fill()
        // 笔身（稍粗更明显）
        ctx.fillStyle = this.brushColor; ctx.fillRect(-bodyLen, -thickness/2, bodyLen - Math.min(4, thickness), thickness)
        ctx.restore()
      }
      const drawMarkerIcon = ()=>{
        ctx.save(); ctx.translate(cx, cy); ctx.rotate(angleDL)
  // 扁平笔头（前沿在 0 处）
  ctx.fillStyle = hi; const h = Math.max(8, thickness*1.5); ctx.fillRect(-bodyLen, -h/2, bodyLen, h)
        ctx.restore()
      }
      const drawPencilIcon = ()=>{
        ctx.save(); ctx.translate(cx, cy); ctx.rotate(angleDL)
        // 木质笔尖
  const tipDepth = Math.min(10, thickness*2.4)
        ctx.fillStyle = '#d4a373'; ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(-tipDepth, thickness/2); ctx.lineTo(-tipDepth, -thickness/2); ctx.closePath(); ctx.fill()
        // 铅芯
  ctx.fillStyle = '#333'; const core = Math.max(1, Math.round(thickness*0.3)); ctx.fillRect(-1, -core/2, 1, core)
        // 笔身
  ctx.fillStyle = this.brushColor; ctx.fillRect(-bodyLen-2, -thickness/2, bodyLen - Math.min(2, thickness*0.4), thickness)
        ctx.restore()
      }
      const drawEraserIcon = ()=>{
        ctx.save(); ctx.translate(cx, cy); ctx.rotate(-Math.PI/8)
        // 以 nib 为橡皮的一个角
        ctx.fillStyle = '#ffffff'; ctx.strokeStyle = lo; ctx.lineWidth = 1.2
        ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(-6, -6); ctx.lineTo(-12, 0); ctx.lineTo(-6, 6); ctx.closePath(); ctx.fill(); ctx.stroke()
        ctx.restore()
      }

      if (this.erasing) drawEraserIcon()
      else if (this.brushStyle === 'marker') drawMarkerIcon()
      else if (this.brushStyle === 'pencil') drawPencilIcon()
      else drawPenIcon()
      return { url: c.toDataURL('image/png'), hx: cx, hy: cy }
    },

    // toggle shape snapping
    toggleShapeSnap(){ this.shapeSnapEnabled = !this.shapeSnapEnabled; this.showHUD(this.shapeSnapEnabled ? '形状优化：开' : '形状优化：关', 900) },

    // Try to beautify the last stroke by recognizing a geometric shape
    maybeBeautifyLastStroke(e) {
      if (!this.shapeSnapEnabled) return
      const s = this.strokes[this.strokes.length - 1]
      if (!s || !s.points || s.points.length < 3) return
      // simplify path to robust corners
  const pts = this.simplifyPath(s.points, this.getSimplifyTolerance(s.points)) // adaptive tolerance in logical px
      if (!pts || pts.length < 2) return
      let shape = this.recognizeShape(pts)
      if (!shape) shape = this.recognizeByTemplates(pts)
      if (!shape) return
      // show preview instead of immediate replace
      this._shapePreview = { shape, color: s.color, size: s.size, style: s.style, index: this.strokes.length - 1 }
      this._previewMoved = false
      this._previewAnchor = e ? { x: e.clientX, y: e.clientY } : null
      clearTimeout(this._shapePreviewTimer)
      this._shapePreviewTimer = setTimeout(()=>{ if (!this._previewMoved) this.commitShapePreview() }, 600)
      this.redraw()
    },
    commitShapePreview() {
      const p = this._shapePreview
      if (!p) return
      const idx = (typeof p.index==='number') ? p.index : (this.strokes.length - 1)
      if (idx < 0 || idx >= this.strokes.length) { this.cancelShapePreview(); return }
      const s = this.strokes[idx]
      // 形状替换前记录撤销快照
      this.pushUndoSnapshot(); this.redoStack = []
      const shaped = { color: p.color || s.color, size: p.size || s.size, eraser: false, style: p.style || s.style, shape: p.shape }
      this.strokes[idx] = shaped
      this._strokesDirty = true
      this.cancelShapePreview(true)
      this.saveAutosave(); this.redraw(); try { this.showHUD('已优化为：' + (p.shape.label || p.shape.type)) } catch(_){}
    },
    cancelShapePreview(silent=false) {
      try { clearTimeout(this._shapePreviewTimer) } catch(_) {}
      this._shapePreviewTimer = null
      if (this._shapePreview) {
        this._shapePreview = null
        if (!silent) this.redraw()
      }
    },
    cancelShapeHold(){
      try { clearTimeout(this._shapeHoldPreviewTimer) } catch(_) {}
      try { clearTimeout(this._shapeHoldCommitTimer) } catch(_) {}
      this._shapeHoldPreviewTimer = null
      this._shapeHoldCommitTimer = null
      this._shapeHoldAnchor = null
    },
    scheduleShapeHold(e){
      // reset timers and schedule preview (300ms) and commit (600ms) while still pressing
      this.cancelShapeHold()
      this._shapeHoldAnchor = { x: e.clientX, y: e.clientY }
      // preview timer
      this._shapeHoldPreviewTimer = setTimeout(()=>{
        if (!this.drawing || this.erasing) return
        // require minimal movement since anchor
        const dx = (this._shapeHoldAnchor && e.clientX - this._shapeHoldAnchor.x) || 0
        const dy = (this._shapeHoldAnchor && e.clientY - this._shapeHoldAnchor.y) || 0
        if (dx*dx + dy*dy > 36) return
        // compute preview from currentStroke points
        const s = this.currentStroke
        if (!s || !s.points || s.points.length < 3) return
  const pts = this.simplifyPath(s.points, this.getSimplifyTolerance(s.points))
        let shape = this.recognizeShape(pts)
        if (!shape) shape = this.recognizeByTemplates(pts)
        if (!shape) return
        this._shapePreview = { shape, color: s.color, size: s.size, style: s.style }
        this._previewAnchor = { x: e.clientX, y: e.clientY }
        this._previewMoved = false
        this.redraw()
      }, 400)
      // commit timer
      this._shapeHoldCommitTimer = setTimeout(()=>{
        if (!this.drawing || this.erasing) return
        // if preview exists and pointer hasn't moved beyond threshold, commit
        const anc = this._previewAnchor || this._shapeHoldAnchor
        if (anc) {
          const dx = e.clientX - anc.x, dy = e.clientY - anc.y
          if (dx*dx + dy*dy > 36) return
        }
        if (this._shapePreview && this._shapePreview.shape) {
          // finalize current stroke as shape
          // 撤销快照，确保仅撤销这一步
          this.pushUndoSnapshot(); this.redoStack = []
          const shaped = { color: this._shapePreview.color, size: this._shapePreview.size, eraser: false, style: this._shapePreview.style, shape: this._shapePreview.shape }
          // replace live stroke by finalizing directly into strokes
          this.strokes.push(shaped)
          this._strokesDirty = true
          this.drawing = false
          this.currentStroke = null
          this._suppressNextPointerUp = true
          this.cancelShapePreview(true)
          this.cancelShapeHold()
          this.saveAutosave(); this.redraw(); try { this.showHUD('已优化为：' + (shaped.shape.label || shaped.shape.type)) } catch(_){}
        }
      }, 1200)
    },

    // Adaptive simplify tolerance based on shape size (suppress corner jitter on small shapes)
    getSimplifyTolerance(points){
      try{
        const bb = this._bbox(points)
        const diag = Math.hypot(bb.w, bb.h) || 1
        const s = this.dbg?.simplifyScale ?? 0.016
        const lo = this.dbg?.minSimplifyPx ?? 2
        const hi = this.dbg?.maxSimplifyPx ?? 8
        return Math.max(lo, Math.min(hi, diag * s))
      }catch(_){ return 2 }
    },

    // Ramer–Douglas–Peucker simplification
    simplifyPath(points, tolerance = 2) {
      const sq = (x)=> x*x
      const distSqSeg = (p, a, b)=>{
        let x=a.x, y=a.y; let dx=b.x-a.x, dy=b.y-a.y
        if (dx!==0 || dy!==0) {
          const t=((p.x-a.x)*dx + (p.y-a.y)*dy)/(dx*dx+dy*dy)
          if (t>1){ x=b.x; y=b.y } else if (t>0){ x=a.x+dx*t; y=a.y+dy*t }
        }
        dx=p.x-x; dy=p.y-y; return dx*dx+dy*dy
      }
      const simplifyDP = (pts, tolSq)=>{
        const last=pts.length-1
        const stack=[[0,last]]; const keep=Array(pts.length).fill(false)
        keep[0]=keep[last]=true
        while(stack.length){
          const [i,j]=stack.pop(); let maxd=0, idx=-1
          for(let k=i+1;k<j;k++){ const d=distSqSeg(pts[k], pts[i], pts[j]); if(d>maxd){maxd=d; idx=k} }
          if (maxd>tolSq){ keep[idx]=true; stack.push([i,idx],[idx,j]) }
        }
        const out=[]; for(let i=0;i<pts.length;i++){ if(keep[i]) out.push(pts[i]) } return out
      }
      return simplifyDP(points, tolerance*tolerance)
    },

    // Shape recognition (coarse but practical)
    recognizeShape(points){
      const n = points.length
      if (n < 2) return null
      const first = points[0], last = points[n-1]
      const bbox = this._bbox(points)
      const diag = Math.hypot(bbox.w, bbox.h) || 1
  const closed = (Math.hypot(last.x-first.x, last.y-first.y) <= diag*(this.dbg?.closedThreshRatio ?? 0.06))
      const len = this._pathLen(points)

      // 1) Line: almost collinear
      if (!closed) {
        const maxDist = this._maxDistToSegment(points, first, last)
        if (maxDist/diag < (this.dbg?.lineDeviationRatio ?? 0.015) && len/diag > 0.2) {
          return { type:'line', p1:first, p2:last, label:'直线' }
        }
      }

      // Circle / Arc using centroid and radius variance
      if (n >= 8) {
        const centroid = this._centroid(points)
        const dists = points.map(p=> Math.hypot(p.x-centroid.x, p.y-centroid.y))
        const r = dists.reduce((a,b)=>a+b,0)/dists.length
        const variance = dists.reduce((a,d)=> a + Math.pow(d-r,2), 0)/dists.length
        const std = Math.sqrt(variance)
        if (std/(r||1) < 0.12) {
          if (closed) return { type:'circle', c:centroid, r, label:'圆' }
          // arc: estimate sweep via angle range
          const angs = points.map(p=> Math.atan2(p.y-centroid.y, p.x-centroid.x))
          let a0 = angs[0], a1 = angs[angs.length-1]
          const sweep = this._angleSweep(angs)
          if (sweep > Math.PI/6 && sweep < Math.PI*1.8) {
            const ccw = this._isCCW(points)
            return { type:'arc', c:centroid, r, a0, a1, ccw, label:'弧' }
          }
        }
      }

      // Polygonal detection via simplified corners
      if (closed && n >= 4) {
        const corners = this._findCorners(points, Math.max(3, diag*0.03))
        const m = corners.length
        if (m === 3) {
          const [a,b,c] = corners
          const sides=[this._dist(a,b), this._dist(b,c), this._dist(c,a)]
          const kind = this._triangleKind(sides)
          return { type:'polygon', p: corners, closed:true, label: kind || '三角形' }
        }
        if (m === 4) {
          const rightish = this._rightAngles(corners, 0.22)
          if (rightish) return { type:'rect', p: corners, label:'矩形' }
          return { type:'polygon', p: corners, closed:true, label:'四边形' }
        }
        if (m >= 5 && m <= 10) {
          if (this._regularPolygonish(corners)) return { type:'polygon', p: corners, closed:true, label:`正${m}边形` }
          return { type:'polygon', p: corners, closed:true, label:`${m}边形` }
        }
      }

      return null
    },

    // Template-based recognition for special symbols (star, heart) using path resampling and normalization
    recognizeByTemplates(points){
      try {
        const N = 64
        const inRes = this._resamplePath(points, N)
        const cen = this._centroid(inRes)
        const norm = inRes.map(p=>({ x:p.x-cen.x, y:p.y-cen.y }))
        // isotropic normalization (mean radius)
        const scale = Math.max(1e-6, this._meanRadius(norm))
        const maxR = inRes.reduce((m,p)=> Math.max(m, Math.hypot(p.x-cen.x, p.y-cen.y)), 0)
        const normed = norm.map(p=>({ x:p.x/scale, y:p.y/scale }))
        // anisotropic normalization (unit half-bbox) to accommodate tall/flat hearts
        const bb = this._bbox(norm)
        const sx = Math.max(1e-6, bb.w/2)
        const sy = Math.max(1e-6, bb.h/2)
        const normedAniso = norm.map(p=>({ x:p.x/sx, y:p.y/sy }))
        const templates = [
          { name:'五角星', gen: this._templateStar.bind(this) },
          { name:'爱心', gen: this._templateHeart.bind(this) }
        ]
        let best=null
        for(const t of templates){
          const tpl = t.gen(N)
          // First try isotropic
          let cand = this._bestAlignError(normed, tpl)
          cand.variant = 'iso'
          // Hearts often come with aspect ratio variations; try anisotropic normalization too
          if (t.name === '爱心') {
            const cand2 = this._bestAlignError(normedAniso, tpl); cand2.variant = 'aniso'
            if (cand2.err < cand.err) cand = cand2
          }
          if (!best || cand.err < best.err) best = { shapeName: t.name, err: cand.err, rot: cand.rot, rev: cand.rev, tpl, variant: cand.variant }
        }
        if (best) {
          if (best.shapeName === '五角星' && best.err < 0.18) {
            // return an exact regular pentagram parameterization
            const R = Math.max(1e-6, maxR)
            const r = R * 0.38196601125 // golden ratio-based inner radius for {5/2}
            return { type:'star', c: cen, R, r, rot: best.rot, label:'正五角星' }
          }
          // Allow a slightly looser match for heart
          const heartThresh = 0.18
          const polyThresh = 0.12
          const allow = (best.shapeName === '爱心') ? heartThresh : polyThresh
          if (best.err < allow) {
            // map template back as polygon when not a star
            const aligned = this._applyAlign(best.tpl, best.rot, best.rev)
            let mapped
            if (best.shapeName === '爱心' && best.variant === 'aniso') {
              // undo anisotropic normalization
              mapped = aligned.map(p=>({ x: p.x*sx + cen.x, y: p.y*sy + cen.y }))
            } else {
              mapped = aligned.map(p=>({ x: p.x*scale + cen.x, y: p.y*scale + cen.y }))
            }
            return { type:'polygon', p: mapped, closed:true, label: best.shapeName }
          }
        }
      } catch(_) {}
      return null
    },
    _resamplePath(pts, n){
      const Ls=[0]; let L=0
      for(let i=1;i<pts.length;i++){ L+=Math.hypot(pts[i].x-pts[i-1].x, pts[i].y-pts[i-1].y); Ls.push(L) }
      if (L<1e-6) return Array(n).fill(pts[0])
      const out=[]
      for(let k=0;k<n;k++){
        const d = k*L/(n-1)
        let i=0; while(i<Ls.length-1 && Ls[i+1] < d) i++
        const t = (d - Ls[i]) / Math.max(1e-6, (Ls[i+1]-Ls[i]))
        const a=pts[i], b=pts[Math.min(i+1, pts.length-1)]
        out.push({ x: a.x + (b.x-a.x)*t, y: a.y + (b.y-a.y)*t })
      }
      return out
    },
    _meanRadius(pts){ let s=0; for(const p of pts){ s+=Math.hypot(p.x,p.y) } return s/pts.length },
    _rotatePts(pts, ang){ const c=Math.cos(ang), s=Math.sin(ang); return pts.map(p=>({ x:p.x*c - p.y*s, y:p.x*s + p.y*c })) },
    _reversePts(pts){ const a=[...pts]; a.reverse(); return a },
    _bestAlignError(a, tpl){
      // brute-force over rotations, direction, and cyclic shifts (to handle different starting indices)
      const rotSteps=12; const shiftSteps=16
      let best={err:Infinity, rot:0, rev:false}
      const variants=[tpl, this._reversePts(tpl)]
      const shift = (arr, s)=>{ const n=arr.length; const out=new Array(n); for(let i=0;i<n;i++){ out[i]=arr[(i+s)%n] } return out }
      for(let ri=0; ri<variants.length; ri++){
        const base=variants[ri]
        for(let i=0;i<rotSteps;i++){
          const rot=i*(Math.PI*2/rotSteps)
          const r0=this._rotatePts(base, rot)
          for(let sj=0; sj<shiftSteps; sj++){
            const r = shift(r0, Math.floor((r0.length * sj) / shiftSteps))
            let err=0
            for(let k=0;k<a.length;k++){ const dx=a[k].x - r[k].x, dy=a[k].y - r[k].y; err += dx*dx + dy*dy }
            err/=a.length
            if (err < best.err) best={ err, rot, rev:(ri===1) }
          }
        }
      }
      return best
    },
    _applyAlign(tpl, rot, rev){ const base = rev ? this._reversePts(tpl) : tpl; return this._rotatePts(base, rot) },
    _templateStar(n){
      // five-point regular star (pentagram) normalized: R=1, r≈0.381966 (golden ratio relation)
      const pts=[]; const R=1, r=0.38196601125
      for(let i=0;i<10;i++){
        const ang = -Math.PI/2 + i*(Math.PI/5)
        const rad = (i%2===0)? R : r
        pts.push({ x: Math.cos(ang)*rad, y: Math.sin(ang)*rad })
      }
      pts.push(pts[0])
      return this._resamplePath(pts, n)
    },
    _templateHeart(n){
      // classic heart param, normalized
      const pts=[]; const K=1/18
      for(let i=0;i<=200;i++){
        const t=i/200 * Math.PI*2
        const x = 16*Math.pow(Math.sin(t),3)
        const y = 13*Math.cos(t) - 5*Math.cos(2*t) - 2*Math.cos(3*t) - Math.cos(4*t)
        pts.push({ x:x*K, y:-y*K })
      }
      return this._resamplePath(pts, n)
    },

    // helpers for recognition
    _bbox(pts){
      let minx=Infinity,miny=Infinity,maxx=-Infinity,maxy=-Infinity
      for(const p of pts){ if(p.x<minx)minx=p.x; if(p.y<miny)miny=p.y; if(p.x>maxx)maxx=p.x; if(p.y>maxy)maxy=p.y }
      return { x:minx, y:miny, w:maxx-minx, h:maxy-miny }
    },
    _pathLen(pts){ let L=0; for(let i=1;i<pts.length;i++) L+=Math.hypot(pts[i].x-pts[i-1].x, pts[i].y-pts[i-1].y); return L },
    _dist(a,b){ return Math.hypot(a.x-b.x,a.y-b.y) },
    _maxDistToSegment(pts, a, b){
      let maxd=0
      for(const p of pts){
        const d=this._pointSegDist(p,a,b); if(d>maxd) maxd=d
      }
      return maxd
    },
    _pointSegDist(p,a,b){
      const vx=b.x-a.x, vy=b.y-a.y
      const wx=p.x-a.x, wy=p.y-a.y
      const c1= vx*wx + vy*wy
      if (c1<=0) return Math.hypot(p.x-a.x,p.y-a.y)
      const c2= vx*vx + vy*vy
      if (c2<=c1) return Math.hypot(p.x-b.x,p.y-b.y)
      const t= c1/c2
      const px=a.x + t*vx, py=a.y + t*vy
      return Math.hypot(p.x-px, p.y-py)
    },
    _centroid(pts){ let sx=0,sy=0; for(const p of pts){ sx+=p.x; sy+=p.y } return { x:sx/pts.length, y:sy/pts.length } },
    _isCCW(pts){ let s=0; for(let i=0;i<pts.length-1;i++){ const a=pts[i], b=pts[i+1]; s += (b.x-a.x)*(b.y+a.y) } return s<0 },
    _angleSweep(angs){
      // unwrap angles and compute total sweep
      let total=0
      for(let i=1;i<angs.length;i++){
        let d = angs[i]-angs[i-1]
        while(d>Math.PI) d-=Math.PI*2
        while(d<-Math.PI) d+=Math.PI*2
        total += Math.abs(d)
      }
      return total
    },
    _findCorners(pts, tol){
      // detect corners; ignore tiny zigzags near joins
      const bb = this._bbox(pts)
      const diag = Math.hypot(bb.w, bb.h) || 1
      const minSeg = Math.max(3, diag * (this.dbg?.cornerMinSegRatio ?? 0.02))
      const out=[pts[0]]
      for(let i=1;i<pts.length-1;i++){
        const a=pts[i-1], b=pts[i], c=pts[i+1]
        const v1={x:b.x-a.x,y:b.y-a.y}, v2={x:c.x-b.x,y:c.y-b.y}
        const l1=Math.hypot(v1.x,v1.y), l2=Math.hypot(v2.x,v2.y)
        if (l1<minSeg || l2<minSeg) continue // ignore jittery micro segments
        const cos=(v1.x*v2.x+v1.y*v2.y)/(l1*l2)
        const ang=Math.acos(Math.max(-1,Math.min(1,cos)))
        const th = (this.dbg?.cornerAngleDeg ?? 33) * Math.PI / 180
        if (ang > th) out.push(b) // significant turn counts as corner
      }
      out.push(pts[pts.length-1])
      // close if near
      const first=out[0], last=out[out.length-1]
      if (Math.hypot(first.x-last.x, first.y-last.y) < tol) out[out.length-1]=first
      // remove near-duplicates and nearly collinear inner points
      const res=[]
      for(const p of out){ if(!res.length || this._dist(res[res.length-1], p) > tol) res.push(p) }
      // prune nearly straight angles to suppress corner jitter
      const prune=[]; if(res.length>2){
        prune.push(res[0])
        for(let i=1;i<res.length-1;i++){
          const A=prune[prune.length-1], B=res[i], C=res[i+1]
          const v1={x:B.x-A.x,y:B.y-A.y}, v2={x:C.x-B.x,y:C.y-B.y}
          const l1=Math.hypot(v1.x,v1.y), l2=Math.hypot(v2.x,v2.y)
          if (l1<minSeg || l2<minSeg) continue
          const cos=(v1.x*v2.x+v1.y*v2.y)/(l1*l2)
          const ang=Math.acos(Math.max(-1,Math.min(1,cos)))
          const pruneTh = (this.dbg?.pruneAngleDeg ?? 10) * Math.PI / 180
          if (ang > pruneTh) prune.push(B) // only keep if angle change big enough
        }
        prune.push(res[res.length-1])
      }
      const finalPts = (prune.length? prune : res)
      if (finalPts.length>2 && this._dist(finalPts[0], finalPts[finalPts.length-1])<tol) finalPts.pop()
      return finalPts
    },
    _rightAngles(corners, tol){
      if (corners.length!==4) return false
      const pts=[...corners, corners[0]]
      for(let i=0;i<4;i++){
        const a=pts[i], b=pts[i+1], c=pts[(i+2)]
        const v1={x:b.x-a.x,y:b.y-a.y}, v2={x:c.x-b.x,y:c.y-b.y}
        const l1=Math.hypot(v1.x,v1.y), l2=Math.hypot(v2.x,v2.y)
        if (l1<1 || l2<1) return false
        const cos=(v1.x*v2.x+v1.y*v2.y)/(l1*l2)
        if (Math.abs(cos) > tol) return false // near 90°
      }
      return true
    },
    _regularPolygonish(corners){
      const n=corners.length
      if (n<5) return false
      const cen=this._centroid(corners)
      const edg=[]; const rad=[]
      for(let i=0;i<n;i++){
        const a=corners[i], b=corners[(i+1)%n]
        edg.push(this._dist(a,b))
        rad.push(this._dist(a,cen))
      }
      const cv=(arr)=>{ const m=arr.reduce((x,y)=>x+y,0)/arr.length; const sd=Math.sqrt(arr.reduce((s,v)=>s+(v-m)*(v-m),0)/arr.length); return sd/(m||1) }
      return cv(edg)<0.25 && cv(rad)<0.25
    },
    _triangleKind(sides){
      const [a,b,c]=sides.sort((x,y)=>x-y)
      const eq=(x,y)=> Math.abs(x-y) <= 0.06*((x+y)/2)
      if (eq(a,b) && eq(b,c)) return '等边三角形'
      if (eq(a,b) || eq(b,c)) return '等腰三角形'
      if (Math.abs(a*a + b*b - c*c) <= 0.12*(a*a + b*b)) return '直角三角形'
      return '三角形'
    },

    // rotation UI: slow rotate handling
    onRotateMouseDown(which, ev) {
      this.slowRotateActive = which // 'cw' or 'ccw'
      // keep hover hint visible while interacting; it is controlled by hover handlers
      this._wheelHandler = (e)=>{
        e.preventDefault()
        const delta = -e.deltaY * 0.15
        const sign = which === 'cw' ? 1 : -1
        this.rotation = (this.rotation + sign * delta + 360) % 360
        this.redraw(); this.updateToolbarPositions()
      }
      window.addEventListener('wheel', this._wheelHandler, { passive: false })
      // stop on mouseup anywhere
      this._mouseUpStop = ()=>{ this.onRotateMouseUp(); window.removeEventListener('mouseup', this._mouseUpStop) }
      window.addEventListener('mouseup', this._mouseUpStop)
    },
    onRotateMouseUp() {
      if (this._wheelHandler) { window.removeEventListener('wheel', this._wheelHandler); this._wheelHandler = null }
      this.slowRotateActive = null
      // hover hint is controlled by mouseenter/leave
    },
    onSlowRotateHover(v){ this.showRotateHoverHint = !!v },
    // export as .zxkjbb zip: board.json + embedded image assets
  async exportBoard() {
    // 自动将所有外部图片内嵌为 dataURL（静默），避免跨域/刷新丢失
    try { await this.embedAllExternalImages(true) } catch(_){ }
      // 1) build manifest and include paste preview (未提交的粘贴内容也导出一次)
      const strokesOut = [...(this.strokes || [])]
      if (this.pasteActive && this.pasteStroke) strokesOut.push(this.pasteStroke)
      const manifest = {
        version: 1,
        meta: { width: this.width, height: this.height, bgColor: this.bgColor, texture: this.texture, rotation: this.rotation, scale: this.scale, panX: this.panX, panY: this.panY },
        strokes: JSON.parse(JSON.stringify(strokesOut)) // shallow clone for src rewrite
      }
      // 2) collect image strokes and embed assets
      const zip = new JSZip()
      const assetsDir = 'assets/images/'
      let imgIndex = 0
      let unresolvedExportImages = 0
      const toImageBlob = (img, mime='image/png')=> new Promise((resolve)=>{
        try{
          const c = document.createElement('canvas'); c.width = img.naturalWidth || img.width; c.height = img.naturalHeight || img.height
          const cctx = c.getContext('2d'); cctx.drawImage(img, 0, 0)
          c.toBlob((blob)=> resolve(blob), mime)
        }catch(_){ resolve(null) }
      })
      const fetchBlobFromSrc = async (src)=>{
        try {
          const res = await fetch(src)
          if (!res.ok) return null
          return await res.blob()
        } catch(_) { return null }
      }
      const pickExtByMime = (mime)=>{
        if (!mime) return '.png'
        const m = mime.toLowerCase()
        if (m.includes('jpeg') || m.includes('jpg')) return '.jpg'
        if (m.includes('webp')) return '.webp'
        if (m.includes('png')) return '.png'
        return '.png'
      }
      const ensureImage = (src)=> new Promise((resolve)=>{
        const im = new Image(); im.crossOrigin = 'anonymous'; im.decoding = 'sync';
        im.onload = ()=> resolve(im)
        im.onerror = ()=> resolve(null)
        im.src = src
      })
      for (let i=0;i<manifest.strokes.length;i++){
        const s = manifest.strokes[i]
        if (s && s.type === 'image' && s.src && typeof s.src === 'string'){
          if (s._importMissing) {
            s.src = 'missing://legacy-image'
            continue
          }
          let imgObj = null
          try { imgObj = (strokesOut[i] && strokesOut[i]._img) ? strokesOut[i]._img : null } catch(_){ }
          if (!imgObj) imgObj = await ensureImage(s.src)
          if (imgObj) {
            let blob = await toImageBlob(imgObj)
            // 如果被跨域污染或 SVG 等导致 toBlob 失败，尝试直接 fetch 原 src（支持 blob:/data:）
            if (!blob && typeof s.src === 'string') {
              blob = await fetchBlobFromSrc(s.src)
            }
            if (blob) {
              const ext = pickExtByMime(blob.type)
              const fname = `${assetsDir}img_${(imgIndex++).toString().padStart(3,'0')}${ext}`
              zip.file(fname, blob)
              s.src = fname
            }
          }
          if (!String(s.src || '').startsWith(assetsDir) && !String(s.src || '').startsWith('data:')) {
            unresolvedExportImages += 1
          }
        }
      }
      if (unresolvedExportImages > 0) {
        const msg = `导出已取消：有 ${unresolvedExportImages} 张图片未能打包进文件。请重新插入这些图片后再导出。`
        try { this.showHUD(msg, 2400) } catch(_){ try { alert(msg) } catch(__){} }
        return
      }
      // 3) add manifest json
      zip.file('board.json', JSON.stringify(manifest))
      // 4) generate and download
      const outBlob = await zip.generateAsync({ type: 'blob' })
      const url = URL.createObjectURL(outBlob)
      const a = document.createElement('a')
      a.href = url
      const safeTitle = String(this.boardTitle || 'whiteboard').trim().replace(/[\\/:*?"<>|]+/g, '_').replace(/\s+/g, '-') || 'whiteboard'
      a.download = `${safeTitle}_${Date.now()}.zxkjbb`
      a.click()
      URL.revokeObjectURL(url)
      try { this.showHUD('导出完成', 1200) } catch(_){ }
    },
    async onImportFile(e) {
      const f = e.target.files && e.target.files[0]
      if (!f) return
      try {
        await this.importFile(f)
      } catch (err) {
        console.error('import failed', err)
        alert('导入失败: 文件格式不受支持或已损坏')
      }
      e.target.value = null
    },
    // create a screenshot (PNG) of the current board
    screenshot() {
      // render to offscreen canvas honoring rotation and current display scale
      const s = (this.displayScale / (this.scale || 1)) * (this.scale || 1) // equals displayScale; kept to avoid NaN
      const rot = ((this.rotation % 360) + 360) % 360
      let outW = Math.round(this.width * s)
      let outH = Math.round(this.height * s)
      if (rot === 90 || rot === 270) {
        ;[outW, outH] = [outH, outW]
      }
      const off = document.createElement('canvas')
      off.width = outW
      off.height = outH
      const octx = off.getContext('2d')
      octx.save()
      // translate to center in pixels
      octx.translate(off.width / 2, off.height / 2)
      octx.rotate(rot * Math.PI / 180)
      octx.scale(s, s)
      octx.translate(-this.width / 2, -this.height / 2)
  // background (entire offscreen also filled to avoid transparency)
  octx.fillStyle = this.bgColor
  octx.fillRect(0,0,off.width,off.height)
      // texture
      if (this.texture === 'grid'){
        octx.save(); octx.strokeStyle='rgba(0,0,0,0.06)'; octx.lineWidth=1
        const step=24, padX=this.width, padY=this.height
        for(let x=-padX;x<=this.width+padX;x+=step){ octx.beginPath(); octx.moveTo(x,-padY); octx.lineTo(x,this.height+padY); octx.stroke() }
        for(let y=-padY;y<=this.height+padY;y+=step){ octx.beginPath(); octx.moveTo(-padX,y); octx.lineTo(this.width+padX,y); octx.stroke() }
        octx.restore()
      }
      // draw strokes (in logical coords; transforms applied)
      for(const sst of this.strokes){
        this.drawStrokeOn(octx, sst)
      }
      octx.restore()
      off.toBlob((blob)=>{
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url; a.download = `whiteboard_${Date.now()}.png`; a.click(); URL.revokeObjectURL(url)
      })
    },

    // undo/redo/clear/autosave helpers
    pushUndoSnapshot() {
      try {
        // 序列化笔划以去除不可序列化的 _img 等临时字段
        const serial = (this.strokes || []).map(s=>{ const { _img, ...rest } = s || {}; return rest })
        this.undoStack.push(JSON.stringify(serial))
        if (this.undoStack.length > 200) this.undoStack.shift()
      } catch (e) { /* ignore */ }
    },
    undo() {
      if (!this.undoStack.length) return
      try {
        const prev = this.undoStack.pop()
        // 将当前状态序列化推入重做栈
        const serial = (this.strokes || []).map(s=>{ const { _img, ...rest } = s || {}; return rest })
        this.redoStack.push(JSON.stringify(serial))
        this.strokes = JSON.parse(prev)
        this._strokesDirty = true
        this.redraw()
        this.saveAutosave()
      } catch (e) { console.error(e) }
    },
    redo() {
      if (!this.redoStack.length) return
      try {
        const next = this.redoStack.pop()
        const serial = (this.strokes || []).map(s=>{ const { _img, ...rest } = s || {}; return rest })
        this.undoStack.push(JSON.stringify(serial))
        this.strokes = JSON.parse(next)
        this._strokesDirty = true
        this.redraw()
        this.saveAutosave()
      } catch (e) { console.error(e) }
    },
    clearCanvas() {
      // record state for undo
      this.pushUndoSnapshot()
      this.redoStack = []
      // clear all strokes
  this.strokes = []
  this._strokesDirty = true
      // reset view position: center, zero rotation, fit whole canvas
      this.rotation = 0
      this.panX = 0
      this.panY = 0
      // if auto-fit is enabled, restore logical size to N×viewport to avoid carrying over prior expansions
      if (this._autoFitContent) {
        const mul = this._autoFitLogicalMultiplier || 1
        this.width = Math.max(1, Math.round(this.viewW * mul))
        this.height = Math.max(1, Math.round(this.viewH * mul))
  this.ensureStrokesLayer()
  this._strokesDirty = true
      }
  // 基于 contain 放大一些，让初始视角更近一点；同时跳过一次扩容，避免立即扩展画布
  const contain = (typeof this.computeContainRotated === 'function') ? this.computeContainRotated() : this.computeContain()
  const closer = contain * (this._resetCloserFactor || 1.5)
  // 不低于 contain，不超过最大显示比例
  const target = Math.max(contain, Math.min(this.maxDisplayScale, closer))
      this._skipCoverageOnce = true
      this.scale = target / (this.computeFit() || 1)
      this.ensureVisibleCoverage()
      this.scale = target / (this.computeFit() || 1)
      // refresh visuals and save
  this.redraw()
      this.updateCursor(); this.updateToolbarPositions()
      this.saveAutosave()
      try { this.showHUD('已清空并复位') } catch(_){}
    },
    saveAutosave() {
      try {
        // 深拷贝 strokes，并将 blob: 源的粘贴图片尽可能转为 dataURL 以便刷新后可恢复
        const strokesOut = (this.strokes || []).map(s=>{
          if (s && s.type==='image' && s.src && typeof s.src==='string' && s.src.startsWith('blob:')){
            try{
              const img = s._img
              if (img && img.complete) {
                const c = document.createElement('canvas'); c.width = img.naturalWidth || img.width; c.height = img.naturalHeight || img.height
                const cctx = c.getContext('2d'); cctx.drawImage(img, 0, 0)
                const data = c.toDataURL('image/png')
                return { ...s, _img: undefined, src: data }
              }
            }catch(_){ }
          }
          const { _img, ...rest } = s || {}
          return rest
        })
  const meta = { width: this.width, height: this.height, bgColor: this.bgColor, texture: this.texture, rotation: this.rotation, scale: this.scale, panX: this.panX, panY: this.panY, eyeProtectActive: !!this.eyeProtectActive, selectedEyeColor: this._selectedEyeColor || '' }
        const extra = {}
        if (this.pasteActive && this.pasteStroke) {
          // 也保存粘贴预览（不提交），下次可继续编辑
          let ps = this.pasteStroke
          if (ps && ps.type==='image' && ps._img && ps._img.complete && ps.src && ps.src.startsWith('blob:')){
            try{
              const img = ps._img
              const c = document.createElement('canvas'); c.width = img.naturalWidth || img.width; c.height = img.naturalHeight || img.height
              const cctx = c.getContext('2d'); cctx.drawImage(img, 0, 0)
              const data = c.toDataURL('image/png')
              ps = { ...ps, _img: undefined, src: data }
            }catch(_){ }
          } else {
            const { _img, ...rest } = ps
            ps = rest
          }
          extra.paste = { active: true, stroke: ps }
        }
        const payload = { meta, strokes: strokesOut, ...extra }
        localStorage.setItem(this.autosaveKey, JSON.stringify(payload))
        this.$emit('autosave-updated', {
          autosaveKey: this.autosaveKey,
          strokeCount: strokesOut.length,
          updatedAt: Date.now(),
          hasPastePreview: !!extra.paste
        })
      } catch (e) { console.error('autosave failed', e) }
    },
    loadAutosave() {
      try {
        const raw = localStorage.getItem(this.autosaveKey)
        if (!raw) return
        const payload = JSON.parse(raw)
        if (payload && payload.strokes) {
          this.width = payload.meta?.width || this.width
          this.height = payload.meta?.height || this.height
          this.bgColor = (payload.meta && 'bgColor' in payload.meta) ? payload.meta.bgColor : this.bgColor
          this.texture = payload.meta?.texture || this.texture
          this.rotation = payload.meta?.rotation || this.rotation
          this.scale = payload.meta?.scale || this.scale
          this.panX = (typeof payload.meta?.panX === 'number') ? payload.meta.panX : 0
          this.panY = (typeof payload.meta?.panY === 'number') ? payload.meta.panY : 0
          // 恢复护眼状态与选择
          if (payload.meta && ('eyeProtectActive' in payload.meta)) this.eyeProtectActive = !!payload.meta.eyeProtectActive
          if (payload.meta && ('selectedEyeColor' in payload.meta)) this._selectedEyeColor = payload.meta.selectedEyeColor || ''
          this.strokes = payload.strokes || []
          // clamp autosaved logical size to current caps
          this.clampLogicalSize(true)
          // 恢复粘贴预览（如有）
          if (payload.paste && payload.paste.active && payload.paste.stroke) {
            this.pasteActive = true
            this.pasteStroke = payload.paste.stroke
          }
          this.resizeCanvas()
          // initialize undo stack with current state (strip transient fields)
          const serial = (this.strokes || []).map(s=>{ const { _img, ...rest } = s || {}; return rest })
          this.undoStack = [JSON.stringify(serial)]
          this.redoStack = []
          // ensure strokes layer is rebuilt from loaded strokes even if canvas size unchanged
          this._strokesDirty = true
          this.redraw && this.redraw()
        }
      } catch (e) { console.error('load autosave failed', e) }
    }
    ,
    // 将所有图像笔划的 src 转为 dataURL（内嵌），以避免跨域/刷新后丢失
  async embedAllExternalImages(quiet=false){
      const isEmbedded = (src)=> typeof src==='string' && src.startsWith('data:')
      const needsEmbed = (src)=> typeof src==='string' && !src.startsWith('data:')
      const toDataURLViaFetch = async (src)=>{
        try{
          const res = await fetch(src)
          if(!res.ok) return null
          const blob = await res.blob()
          return await new Promise((resolve)=>{ const fr=new FileReader(); fr.onload=()=>resolve(fr.result); fr.onerror=()=>resolve(null); fr.readAsDataURL(blob) })
        }catch(_){ return null }
      }
      const toDataURLViaCanvas = async (img, preferMime)=>{
        try{
          if(!img) return null
          // ensure image is loaded
          if(!img.complete){ await new Promise((r)=>{ img.onload=()=>r(); img.onerror=()=>r() }) }
          const iw = img.naturalWidth || img.width, ih = img.naturalHeight || img.height
          if(!iw || !ih) return null
          const c = document.createElement('canvas'); c.width=iw; c.height=ih
          const cx = c.getContext('2d'); cx.drawImage(img,0,0)
          // try keep type; fallback to png
          let mime = 'image/png'
          if (preferMime && /^image\/(png|jpeg|webp)/i.test(preferMime)) mime = preferMime
          try { return c.toDataURL(mime, mime.includes('jpeg') ? 0.92 : undefined) } catch(_){ return c.toDataURL('image/png') }
        }catch(_){ return null }
      }
      const ensureImage = (src)=> new Promise((resolve)=>{
        const im = new Image(); im.crossOrigin='anonymous'; im.decoding='sync';
        im.onload=()=>resolve(im); im.onerror=()=>resolve(null); im.src=src
      })

      let total=0, converted=0, failed=0
      // 处理 strokes
      for (let i=0;i<(this.strokes||[]).length;i++){
        const s = this.strokes[i]
        if (!s || s.type!=='image' || !s.src || !needsEmbed(s.src)) continue
        total++
        let dataURL = await toDataURLViaFetch(s.src)
        if (!dataURL){
          let imgObj = null
          try { imgObj = s._img || null } catch(_){ }
          if(!imgObj) imgObj = await ensureImage(s.src)
          dataURL = await toDataURLViaCanvas(imgObj)
        }
        if (dataURL){
          const { _img, ...rest } = s
          this.strokes[i] = { ...rest, src: dataURL }
          converted++
        } else {
          failed++
        }
      }
      // 处理粘贴预览（如有）
      if (this.pasteActive && this.pasteStroke && this.pasteStroke.type==='image'){
        const s = this.pasteStroke
        if (s.src && needsEmbed(s.src)){
          let dataURL = await toDataURLViaFetch(s.src)
          if (!dataURL){
            let imgObj = null
            try { imgObj = s._img || null } catch(_){ }
            if(!imgObj) imgObj = await ensureImage(s.src)
            dataURL = await toDataURLViaCanvas(imgObj)
          }
          if (dataURL){ this.pasteStroke = { ...s, _img: undefined, src: dataURL }; converted++; total++ } else { failed++; total++ }
        }
      }
      if (total>0){ this._strokesDirty = true; this.redraw(); this.saveAutosave() }
      if (!quiet) {
        const msg = total===0 ? '没有需要内嵌的图片' : `已内嵌图片 ${converted}/${total}` + (failed?`，失败 ${failed}`:'')
        try { this.showHUD(msg) } catch(_){ try{ !quiet && alert(msg) }catch(__){} }
      }
    }
  }
}
</script>

<style scoped>
.whiteboard-root{ display:flex; height:100%; min-height:600px; position:relative; }
.whiteboard-root{ display:flex; height:100%; min-height:600px; position:relative; }
.wb-left{ width:64px; display:flex; flex-direction:column; gap:10px; padding:10px; align-items:center; background:transparent }
.wb-btn{ width:44px; height:44px; display:inline-flex; align-items:center; justify-content:center; border-radius:10px; background:rgba(15,23,42,0.8); color:#fff; border:1px solid rgba(255,255,255,0.08); box-shadow:0 8px 24px rgba(0,0,0,0.35); cursor:pointer; backdrop-filter: blur(6px) }
.wb-sep{ height:1px; width:36px; background:rgba(255,255,255,0.04); margin:8px 0 }
.wb-color input{ width:36px; height:36px; border-radius:6px; border:none; padding:0 }
.round-color{ width:36px; height:36px; border-radius:50%; border:none; padding:0 }
.brush-color .round-color{ box-shadow:none }
.wb-canvas-area{ flex:1; display:flex; align-items:stretch; justify-content:stretch; padding:0; height:100vh }
.wb-canvas-wrap{ width:100%; height:100%; display:flex; align-items:stretch; justify-content:stretch; overflow:hidden; background:transparent }
.wb-canvas{ background:transparent; box-shadow:0 6px 30px rgba(2,6,23,0.6); border-radius:6px; display:block; width:100%; height:100% }
.wb-right{ width:110px; display:flex; flex-direction:column; gap:10px; padding:10px; align-items:center }
.wb-right.wb-right-wide{ width:260px }
.wb-bottom{ position:fixed; left:50%; transform:translateX(-50%); bottom:12px; background:linear-gradient(90deg, rgba(15,23,42,0.85), rgba(15,23,42,0.7)); padding:10px 18px; border-radius:12px; border:1px solid rgba(255,255,255,0.08); box-shadow:0 8px 24px rgba(0,0,0,0.4); backdrop-filter: blur(6px) }
.brush-controls{ max-width:1000px; margin:0 auto; display:flex; align-items:center; gap:18px }
.brush-group, .eraser-group, .color-group{ display:flex; flex-direction:column; align-items:center; gap:6px }
.brush-group .brush-style{ display:flex; align-items:center }
.eraser-group .tool{ margin-bottom:4px }
.brush-style .tool{ margin-right:6px; padding:6px 8px; border-radius:6px; background:rgba(255,255,255,0.02); color:#fff }
.brush-style .tool.active{ box-shadow:0 6px 18px rgba(0,0,0,0.45); transform:translateY(-2px) }
.brush-size input[type=range]{ width:160px }
.wb-color-palette, .color-palette{ display:flex; gap:6px; align-items:center; margin-top:6px }
.swatch{ width:22px; height:22px; border-radius:6px; cursor:pointer; border:1px solid rgba(0,0,0,0.06) }
.swatch[title]{ box-shadow:0 2px 6px rgba(0,0,0,0.12) }
.eye-btn{ margin-left:6px; padding:4px 8px; border-radius:6px; background:rgba(255,255,255,0.04); color:#fff; border:1px solid rgba(255,255,255,0.06); cursor:pointer }
.eye-open{ margin-left:4px; padding:4px 6px; border-radius:6px; background:rgba(255,255,255,0.02); color:#fff; border:1px solid rgba(255,255,255,0.04); cursor:pointer }
.eye-palette{ display:flex; gap:6px; margin-top:6px; }
.size-ind{ display:inline-block; width:36px; text-align:center }
.brush-color input{ width:36px; height:36px; border-radius:6px; border:none }
/* round color inputs */
.round-color, .round-bg-color { width:36px; height:36px; border-radius:50%; border:none; padding:0; cursor:pointer }
.round-bg-color { box-shadow:0 6px 18px rgba(0,0,0,0.25); }
/* black/white swatches for brush */
.bw-group{ display:flex; flex-direction:column; gap:8px; margin-left:8px }
.bw-swatch{ width:28px; height:28px; border-radius:50%; border:1px solid rgba(0,0,0,0.06); box-shadow:0 2px 6px rgba(0,0,0,0.12); cursor:pointer }
/* recent colors row */
.brush-side-row{ display:flex; align-items:center; gap:12px; margin-top:8px }
.recent-row{ display:flex; gap:8px; align-items:center }
.recent-swatch{ width:22px; height:22px; border-radius:50%; border:1px solid rgba(0,0,0,0.06); cursor:pointer }
.recent-placeholder{ width:22px; height:22px; border-radius:50%; border:2px dashed rgba(255,255,255,0.12); background:transparent }
.hidden-file{ display:none }

/* 文本样式面板 */
.text-style-panel{ width:100%; margin-top:8px; padding:10px; border-radius:10px; background:rgba(15,23,42,0.6); border:1px solid rgba(255,255,255,0.08); box-shadow:0 6px 18px rgba(0,0,0,0.35); color:#e6eef8 }
.text-style-panel .panel-title{ font-weight:700; margin-bottom:8px; font-size:13px }
.text-style-panel .row{ display:flex; align-items:center; gap:8px; margin-bottom:8px }
.text-style-panel label{ width:44px; font-size:12px; opacity:0.9 }
.text-style-panel select{ flex:1; height:28px; background:#0f1720; color:#e6eef8; border:1px solid rgba(255,255,255,0.06); padding:0 8px; border-radius:6px }
.text-style-panel input[type=color]{ width:28px; height:28px; border:none; padding:0; border-radius:6px; box-shadow:0 2px 6px rgba(0,0,0,0.2) }
.text-style-panel input[type=range]{ flex:1 }
.text-style-panel .val{ width:44px; text-align:right; font-size:12px; opacity:0.9 }
.text-style-panel .align-group{ display:flex; gap:6px }
.text-style-panel .align-group button{ width:32px; height:28px; border-radius:6px; background:rgba(255,255,255,0.06); color:#e6eef8; border:1px solid rgba(255,255,255,0.08); cursor:pointer }
.text-style-panel .align-group button.active{ background:rgba(59,130,246,0.28); border-color:rgba(59,130,246,0.5) }

/* ensure controls readable */
.wb-left, .wb-right, .wb-bottom, .wb-btn, .tool{ color:#e6eef8 }

/* texture select contrast */
.texture-wrap{ width:64px; display:flex; flex-direction:column; align-items:center; gap:6px; padding:6px 0 }
.texture-caption{ font-size:12px; opacity:1; user-select:none }
.texture-select{ width:92px; height:28px; background:#0f1720; color:#e6eef8; border:1px solid rgba(255,255,255,0.06); padding:0 8px; border-radius:6px }

/* position helpers when attached to canvas */
.wb-left{ transition: top 160ms ease, left 160ms ease }
.wb-right{ transition: top 160ms ease, left 160ms ease }
.wb-bottom{ transition: top 160ms ease, left 160ms ease, width 160ms ease }

/* fullscreen button on canvas */
.wb-fullscreen-btn{ position:absolute; right:56px; top:10px; width:40px; height:40px; border-radius:10px; background:rgba(15,23,42,0.85); border:1px solid rgba(255,255,255,0.08); display:inline-flex; align-items:center; justify-content:center; color:#fff; z-index:30; box-shadow:0 8px 24px rgba(0,0,0,0.35); backdrop-filter: blur(6px) }
.wb-exit-page-btn{ position:absolute; right:10px; top:10px; width:40px; height:40px; border-radius:10px; background:rgba(15,23,42,0.85); border:1px solid rgba(255,255,255,0.08); display:inline-flex; align-items:center; justify-content:center; color:#fff; z-index:30; box-shadow:0 8px 24px rgba(0,0,0,0.35); backdrop-filter: blur(6px) }
.wb-exit-btn{ position:absolute; left:10px; top:10px; width:40px; height:40px; border-radius:10px; background:rgba(15,23,42,0.85); border:1px solid rgba(255,255,255,0.08); display:inline-flex; align-items:center; justify-content:center; color:#fff; z-index:30; box-shadow:0 8px 24px rgba(0,0,0,0.35); backdrop-filter: blur(6px) }

/* rotate vertical control */
.rotate-vertical{ display:flex; flex-direction:column; align-items:center; gap:10px; }
.rotate-vertical .wb-btn{ width:44px; height:44px }
.slow-rotate-wrap{ display:flex; flex-direction:column; align-items:center; gap:6px }
.slow-rotate-wrap{ position:relative }
.rotate-hover-hint{ font-size:12px; opacity:0; transition:opacity .2s ease; user-select:none; position:absolute }
.rotate-hover-hint.right{ left:100%; top:50%; transform: translate(8px, -50%); white-space:nowrap }
.rotate-hover-hint.show{ opacity:.95 }

/* HUD */
.wb-hud{ position:fixed; left:50%; transform:translateX(-50%); top:18px; z-index:9999; background:rgba(17,24,39,0.86); color:#f8fafc; padding:8px 14px; border-radius:999px; box-shadow:0 6px 24px rgba(2,6,23,0.6); font-size:13px; opacity:0; transition:opacity .18s ease, transform .12s ease }
.wb-hud.show{ opacity:1; transform:translateX(-50%) translateY(0) }
.wb-hud small{ opacity:0.86; font-size:12px }

/* make brush cursor change smooth */
.wb-canvas { cursor: crosshair }

/* make whiteboard root fill the viewport */
.whiteboard-root{ position:fixed; inset:0; z-index:40; background:transparent }
</style>
