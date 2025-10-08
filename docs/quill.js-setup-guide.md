# Quill.js 설치 및 커스터마이징 가이드

Rails 8.0.2 프로젝트에 Quill.js 2.0.3 리치 텍스트 에디터를 설치하고 Font Size 드롭다운을 커스터마이징하는 완전한 가이드입니다.

## 참고 자료
- [Quill.js 공식 문서](https://quilljs.com/)
- [Quill.js 설치 가이드](https://quilljs.com/docs/installation)
- [Quill.js 툴바 모듈](https://quilljs.com/docs/modules/toolbar)
- [Rails Importmap 가이드](https://github.com/rails/importmap-rails)
- [Stimulus 컨트롤러 가이드](https://stimulus.hotwired.dev/)

## 목차
1. [기본 설치](#기본-설치)
2. [Stimulus 컨트롤러 구현](#stimulus-컨트롤러-구현)
3. [CSS 스타일링](#css-스타일링)
4. [오류 해결](#오류-해결)
5. [사용법](#사용법)

## 기본 설치

### 1. Importmap에 Quill.js 추가

`config/importmap.rb` 파일에 Quill.js CDN을 추가합니다:

```ruby
# config/importmap.rb
pin "application"
pin "@hotwired/turbo-rails", to: "turbo.min.js"
pin "@hotwired/stimulus", to: "stimulus.min.js"
pin "@hotwired/stimulus-loading", to: "stimulus-loading.js"
pin_all_from "app/javascript/controllers", under: "controllers"

# Quill.js 2.0.3 추가
pin "quill", to: "https://cdn.jsdelivr.net/npm/quill@2.0.3/dist/quill.js"
```

### 2. Application.js에 Import 추가

`app/javascript/application.js` 파일에 Quill import를 추가합니다:

```javascript
// app/javascript/application.js
import "@hotwired/turbo-rails"
import "controllers"
import "quill"
```

### 3. CSS 스타일시트 추가

`app/assets/stylesheets/application.css` 파일에 Quill 2.0.3 테마 CSS를 추가합니다:

```css
/* app/assets/stylesheets/application.css */
@import url('https://cdn.jsdelivr.net/npm/quill@2.0.3/dist/quill.snow.css');
```

## Stimulus 컨트롤러 구현

### Quill 컨트롤러 생성

`app/javascript/controllers/quill_controller.js` 파일을 생성하고 다음 코드를 추가합니다:

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["editor", "hiddenInput"]
  static values = {
    placeholder: String,
    readonly: Boolean
  }

  connect() {
    // Wait for Quill to be available globally
    if (typeof window.Quill !== 'undefined') {
      this.initializeQuill()
      this.setupFontSizes()
    } else {
      // Wait a bit for the library to load
      setTimeout(() => {
        if (typeof window.Quill !== 'undefined') {
          this.initializeQuill()
          this.setupFontSizes()
        } else {
          console.error('Quill library not found')
        }
      }, 100)
    }
  }

  disconnect() {
    if (this.quill) {
      this.quill = null
    }
  }

  initializeQuill() {
    const toolbarOptions = [
      [{ 'size': ['8px', '10px', '12px', '14px', '16px', '18px', '24px', '32px', '48px', '72px'] }],
      ['bold', 'italic', 'underline', 'strike'],
      [{ 'color': [] }, { 'background': [] }],
      [{ 'list': 'ordered' }, { 'list': 'bullet' }],
      [{ 'script': 'sub' }, { 'script': 'super' }],
      [{ 'indent': '-1' }, { 'indent': '+1' }],
      [{ 'direction': 'rtl' }],
      [{ 'align': [] }],
      ['blockquote', 'code-block'],
      ['link', 'image'],
      ['clean']
    ]

    this.quill = new window.Quill(this.editorTarget, {
      modules: {
        toolbar: toolbarOptions
      },
      theme: 'snow',
      placeholder: this.placeholderValue || 'Start typing...',
      readOnly: this.readonlyValue || false
    })

    // Update hidden input when content changes
    this.quill.on('text-change', () => {
      if (this.hasHiddenInputTarget) {
        this.hiddenInputTarget.value = this.quill.root.innerHTML
      }
    })

    // Load initial content from hidden input
    if (this.hasHiddenInputTarget && this.hiddenInputTarget.value) {
      this.quill.root.innerHTML = this.hiddenInputTarget.value
    }
  }

  setupFontSizes() {
    // Register custom size attributor
    const SizeStyle = window.Quill.import('attributors/style/size')
    SizeStyle.whitelist = ['8px', '10px', '12px', '14px', '16px', '18px', '24px', '32px', '48px', '72px']
    window.Quill.register(SizeStyle, true)

    // Add CSS for dropdown options to show actual font sizes
    this.addFontSizeStyles()
  }

  addFontSizeStyles() {
    const style = document.createElement('style')
    style.textContent = `
      /* Quill Editor Default Font Size */
      .ql-editor {
        font-size: 16px !important;
      }
      
      /* Font Size Dropdown Label */
      .ql-size .ql-picker-label {
        position: relative;
        min-width: 80px;
      }
      
      .ql-size .ql-picker-label::before {
        content: "Font Size" !important;
        font-size: 14px !important;
        color: #444 !important;
        font-weight: bold !important;
      }
      
      /* Hide the original data-value content and show Font Size */
      .ql-size .ql-picker-label[data-value]::before {
        content: "Font Size" !important;
      }
      
      .ql-size .ql-picker-label[data-value=""]::before {
        content: "Font Size" !important;
      }
      
      /* Hide the original label content */
      .ql-size .ql-picker-label .ql-picker-item {
        display: none !important;
      }
      
      /* Font Size Dropdown Options Container */
      .ql-size .ql-picker-options {
        font-family: Arial, sans-serif !important;
        max-height: 200px !important;
        overflow-y: auto !important;
        border: 1px solid #ccc !important;
        border-radius: 4px !important;
        box-shadow: 0 2px 8px rgba(0,0,0,0.1) !important;
        background: white !important;
        z-index: 1000 !important;
        min-width: 120px !important;
      }
      
      /* Custom scrollbar for webkit browsers */
      .ql-size .ql-picker-options::-webkit-scrollbar {
        width: 8px !important;
      }
      
      .ql-size .ql-picker-options::-webkit-scrollbar-track {
        background: #f1f1f1 !important;
        border-radius: 4px !important;
      }
      
      .ql-size .ql-picker-options::-webkit-scrollbar-thumb {
        background: #c1c1c1 !important;
        border-radius: 4px !important;
      }
      
      .ql-size .ql-picker-options::-webkit-scrollbar-thumb:hover {
        background: #a8a8a8 !important;
      }
      
      /* Font Size Options Styling */
      .ql-size .ql-picker-item {
        padding: 8px 12px !important;
        cursor: pointer !important;
        transition: background-color 0.2s ease !important;
        border-bottom: 1px solid #f0f0f0 !important;
        text-align: left !important;
      }
      
      .ql-size .ql-picker-item:hover {
        background-color: #f5f5f5 !important;
      }
      
      .ql-size .ql-picker-item:last-child {
        border-bottom: none !important;
      }
      
      .ql-size .ql-picker-item[data-value="8px"] {
        font-size: 8px !important;
      }
      .ql-size .ql-picker-item[data-value="8px"]::before {
        content: "8px" !important;
        font-size: 8px !important;
      }
      
      .ql-size .ql-picker-item[data-value="10px"] {
        font-size: 10px !important;
      }
      .ql-size .ql-picker-item[data-value="10px"]::before {
        content: "10px" !important;
        font-size: 10px !important;
      }
      
      .ql-size .ql-picker-item[data-value="12px"] {
        font-size: 12px !important;
      }
      .ql-size .ql-picker-item[data-value="12px"]::before {
        content: "12px" !important;
        font-size: 12px !important;
      }
      
      .ql-size .ql-picker-item[data-value="14px"] {
        font-size: 14px !important;
      }
      .ql-size .ql-picker-item[data-value="14px"]::before {
        content: "14px" !important;
        font-size: 14px !important;
      }
      
      .ql-size .ql-picker-item[data-value="16px"] {
        font-size: 16px !important;
      }
      .ql-size .ql-picker-item[data-value="16px"]::before {
        content: "16px" !important;
        font-size: 16px !important;
      }
      
      .ql-size .ql-picker-item[data-value="18px"] {
        font-size: 18px !important;
      }
      .ql-size .ql-picker-item[data-value="18px"]::before {
        content: "18px" !important;
        font-size: 18px !important;
      }
      
      .ql-size .ql-picker-item[data-value="24px"] {
        font-size: 24px !important;
      }
      .ql-size .ql-picker-item[data-value="24px"]::before {
        content: "24px" !important;
        font-size: 24px !important;
      }
      
      .ql-size .ql-picker-item[data-value="32px"] {
        font-size: 32px !important;
      }
      .ql-size .ql-picker-item[data-value="32px"]::before {
        content: "32px" !important;
        font-size: 32px !important;
      }
      
      .ql-size .ql-picker-item[data-value="48px"] {
        font-size: 48px !important;
      }
      .ql-size .ql-picker-item[data-value="48px"]::before {
        content: "48px" !important;
        font-size: 48px !important;
      }
      
      .ql-size .ql-picker-item[data-value="72px"] {
        font-size: 72px !important;
      }
      .ql-size .ql-picker-item[data-value="72px"]::before {
        content: "72px" !important;
        font-size: 72px !important;
      }
      
      .ql-size .ql-picker-item[data-value=""] {
        font-size: 16px !important;
      }
      .ql-size .ql-picker-item[data-value=""]::before {
        content: "Default (16px)" !important;
        font-size: 16px !important;
      }
    `

    if (!document.querySelector('#quill-font-size-styles')) {
      style.id = 'quill-font-size-styles'
      document.head.appendChild(style)
    }
  }

  // Public method to get content
  getContent() {
    return this.quill ? this.quill.getContents() : null
  }

  // Public method to set content
  setContent(delta) {
    if (this.quill) {
      this.quill.setContents(delta)
    }
  }

  // Public method to get HTML
  getHTML() {
    return this.quill ? this.quill.root.innerHTML : ''
  }

  // Public method to get text
  getText() {
    return this.quill ? this.quill.getText() : ''
  }
}
```

## CSS 스타일링

### 주요 커스터마이징 포인트

1. **기본 폰트 크기 설정**: 에디터의 기본 폰트 크기를 16px로 설정
2. **드롭다운 레이블**: "Font Size"로 고정 표시
3. **스크롤 가능한 드롭다운**: 200px 최대 높이로 제한하고 스크롤 추가
4. **실제 크기 미리보기**: 각 폰트 크기 옵션을 실제 크기로 표시
5. **사용자 경험 개선**: 호버 효과, 커스텀 스크롤바, 적절한 패딩

## 오류 해결

### 1. Quill import 오류
**문제**: `does not provide an export named 'default'` 오류 발생

**해결책**: ES6 import 대신 global window object 사용
```javascript
// 잘못된 방법
import Quill from "quill"

// 올바른 방법
new window.Quill(element, options)
```

### 2. 툴바가 렌더링되지 않는 문제
**원인**: Quill 라이브러리가 로드되기 전에 초기화 시도

**해결책**: setTimeout을 사용하여 라이브러리 로드 대기
```javascript
connect() {
  if (typeof window.Quill !== 'undefined') {
    this.initializeQuill()
  } else {
    setTimeout(() => {
      if (typeof window.Quill !== 'undefined') {
        this.initializeQuill()
      }
    }, 100)
  }
}
```

### 3. 폰트 크기 드롭다운에 잘못된 레이블 표시
**문제**: "Font Size" 대신 선택된 값(예: "8px")이 표시됨

**해결책**: CSS ::before pseudo-element로 강제 오버라이드
```css
.ql-size .ql-picker-label[data-value]::before {
  content: "Font Size" !important;
}
```

## 사용법

### 기본 사용법

```erb
<div data-controller="quill">
  <div data-quill-target="editor"></div>
  <input type="hidden" data-quill-target="hiddenInput" name="content" />
</div>
```

### 옵션 설정

```erb
<div data-controller="quill" 
     data-quill-placeholder-value="여기에 내용을 입력하세요..."
     data-quill-readonly-value="false">
  <div data-quill-target="editor"></div>
  <input type="hidden" data-quill-target="hiddenInput" name="content" />
</div>
```

### 폼과 통합

```erb
<%= form_with model: @article do |form| %>
  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>
  
  <div class="field">
    <%= form.label :content %>
    <div data-controller="quill">
      <div data-quill-target="editor"></div>
      <%= form.hidden_field :content, data: { quill_target: "hiddenInput" } %>
    </div>
  </div>
  
  <%= form.submit %>
<% end %>
```

### JavaScript에서 접근

```javascript
// Stimulus 컨트롤러에서
const content = this.getContent()
const html = this.getHTML()
const text = this.getText()

this.setContent(deltaContent)
```

## 기능 특징

- **8px부터 72px까지 폰트 크기 선택 가능**
- **각 크기 옵션을 실제 크기로 미리보기**
- **16px 기본 폰트 크기**
- **스크롤 가능한 드롭다운 (최대 200px 높이)**
- **"Font Size" 고정 레이블**
- **Rails 폼과 완전 통합**
- **반응형 디자인**
- **읽기 전용 모드 지원**

이 가이드를 따라하면 Rails 8.0.2 프로젝트에 완전히 커스터마이징된 Quill.js 에디터를 성공적으로 설치할 수 있습니다.