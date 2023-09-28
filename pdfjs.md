<!--
 * @Author: please
 * @Date: 2023-09-21 11:40:39
 * @LastEditors: please
 * @LastEditTime: 2023-09-21 13:16:19
 * @Description: 请填写简介
-->
/*
 * @Author: please
 * @Date: 2023-09-21 11:40:39
 * @LastEditors: please
 * @LastEditTime: 2023-09-21 11:40:43
 * @Description: 请填写简介
 */
pdf.js是一个JavaScript库，用于在Web浏览器中解析和渲染PDF文件。要实现内容文本高亮，可以使用pdf.js的自定义渲染器功能。以下是一个简单的示例：

1. 首先，确保已经安装了pdf.js库。可以使用npm进行安装：

```bash
npm install pdfjs-dist
```

2. 创建一个HTML文件，例如`highlightText.html`，并添加以下内容：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF Text Highlighting</title>
    <style>
        #pageContent {
            margin-top: 50px;
        }
    </style>
</head>
<body>
    <div id="pageContent"></div>
    <script src="highlightText.js"></script>
</body>
</html>
```

3. 创建一个JavaScript文件，例如`highlightText.js`，并添加以下内容：

```javascript
const url = 'path/to/your/pdf/file.pdf'; // 替换为你的PDF文件路径
const canvas = document.getElementById('pageContent');
const ctx = canvas.getContext('2d');

pdfjsLib.getDocument(url).promise.then((pdfDoc) => {
  const pageNumber = 1; // 从第一页开始
  pdfDoc.getPage(pageNumber).then((page) => {
    const viewport = page.getViewport({ scale: 1 });
    canvas.height = viewport.height;
    canvas.width = viewport.width;

    const renderContext = {
      canvasContext: ctx,
      viewport: viewport,
    };

    page.render(renderContext).promise.then(() => {
      // 在这里实现文本高亮逻辑
      // 例如，可以遍历页面上的文本元素，根据需要设置不同的背景颜色
      const textLayer = page.getTextContent().items.filter((item) => item.type === 'text');
      textLayer.forEach((textItem) => {
        if (textItem.str === '需要高亮的文本') { // 根据需要修改文本内容
          textItem.set({
            color: 'yellow',
          });
        }
      });

      // 将处理后的页面渲染到canvas上
      const renderTask = page.render(renderContext);
      renderTask.promise.then(() => {
        console.log('文本高亮完成');
      });
    });
  });
});
```

4. 将`path/to/your/pdf/file.pdf`替换为你要处理的PDF文件的路径。

5. 在浏览器中打开`highlightText.html`文件，查看文本高亮效果。

注意：这个示例仅提供了一个简单的文本高亮实现，你可以根据需要修改代码以实现更复杂的文本高亮效果。



PDF.js是一个JavaScript库，用于在Web浏览器中解析和渲染PDF文件。要在Web应用程序中使用PDF.js，请按照以下步骤操作：

1. 下载PDF.js库：访问PDF.js的GitHub发布页面（https://github.com/mozilla/pdf.js/releases）并下载最新版本的PDF.js库。

2. 将PDF.js库添加到项目中：将下载的PDF.js库解压缩并将其中的`web/pdf.worker.js`和`web/viewer.html`文件复制到您的Web项目的目录中。

3. 在HTML文件中引用PDF.js库：在您的HTML文件中，使用`<script>`标签引用`viewer.html`文件。例如：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF.js Example</title>
    <script src="path/to/pdf.js"></script>
</head>
<body>
    <canvas id="pdf-canvas"></canvas>
    <script src="path/to/main.js"></script>
</body>
</html>
```

4. 编写JavaScript代码来加载和显示PDF文件：在`main.js`文件中，编写以下代码以加载和显示PDF文件：

```javascript
const url = 'path/to/your/pdf/file.pdf'; // 替换为您的PDF文件路径
const canvas = document.getElementById('pdf-canvas');
const ctx = canvas.getContext('2d');

PDFJS.getDocument(url).then((pdf) => {
  const totalPages = pdf.numPages;
  let currentPage = 1;

  function renderPage(pageNum) {
    PDFJS.getPage(pageNum).then((page) => {
      const viewport = page.getViewport({ scale: 1 });
      canvas.height = viewport.height;
      canvas.width = viewport.width;

      const renderContext = {
        canvasContext: ctx,
        viewport: viewport,
      };

      page.render(renderContext).then(() => {
        // 在这里添加您需要执行的其他操作，例如处理页面上的交互等
      });
    });
  }

  renderPage(currentPage);

  // 监听翻页事件
  document.getElementById('prev-page').addEventListener('click', () => {
    if (currentPage > 1) {
      currentPage--;
      renderPage(currentPage);
    }
  });

  document.getElementById('next-page').addEventListener('click', () => {
    if (currentPage < totalPages) {
      currentPage++;
      renderPage(currentPage);
    }
  });
});
```

5. 添加HTML元素以控制翻页：在HTML文件中，添加两个按钮元素以允许用户向前翻页和向后翻页。例如：

```html
<button id="prev-page">上一页</button>
<button id="next-page">下一页</button>
```

现在，当用户点击“上一页”或“下一页”按钮时，PDF文件将在Web浏览器中逐页加载和显示。您可以根据需要修改`main.js`文件中的代码以实现其他功能，例如处理页面上的交互等。


要在 Vue 项目中使用 pdf.js，你需要按照以下步骤操作：

1. 首先，安装 pdf.js：

```bash
npm install pdfjs-dist
```

2. 在 Vue 组件中引入 pdf.js：

```javascript
import * as pdfjsLib from 'pdfjs-dist';
```

3. 创建一个 Vue 组件，用于显示 PDF 文件：

```html
<template>
  <div>
    <canvas ref="pdfCanvas"></canvas>
  </div>
</template>

<script>
import * as pdfjsLib from 'pdfjs-dist';

export default {
  data() {
    return {
      pdfDoc: null,
      pageNum: 1,
      pageRendering: false,
      pageNumPending: null,
      scale: 1,
      canvas: null,
      ctx: null,
    };
  },
  mounted() {
    this.canvas = this.$refs.pdfCanvas;
    this.ctx = this.canvas.getContext('2d');
    this.loadPdf('path/to/your/pdf/file.pdf');
  },
  methods: {
    loadPdf(url) {
      pdfjsLib.getDocument(url).promise.then((pdfDoc_) => {
        this.pdfDoc = pdfDoc_;
        this.renderPage(this.pageNum);
      });
    },
    renderPage(num) {
      this.pageRendering = true;
      this.pdfDoc.getPage(num).then((page) => {
        const viewport = page.getViewport({ scale: this.scale });
        this.canvas.height = viewport.height;
        this.canvas.width = viewport.width;

        const renderContext = {
          canvasContext: this.ctx,
          viewport: viewport,
        };
        const renderTask = page.render(renderContext);

        renderTask.promise.then(() => {
          this.pageRendering = false;
          if (this.pageNumPending !== null) {
            this.renderPage(this.pageNumPending);
            this.pageNumPending = null;
          }
        });
      });
    },
  },
};
</script>
```

4. 在上述代码中，替换 `'path/to/your/pdf/file.pdf'` 为你要显示的 PDF 文件的路径。

5. 现在，你可以在其他 Vue 组件中使用这个自定义组件来显示 PDF 文件。例如，在 App.vue 中：

```html
<template>
  <div id="app">
    <PDFViewer />
  </div>
</template>

<script>
import PDFViewer from './components/PDFViewer.vue';

export default {
  components: {
    PDFViewer,
  },
};
</script>
```

这样，你就可以在 Vue 项目中使用 pdf.js 了。





要在 Vue 中使用 PDF.js 渲染 PDF 文件，你可以按照以下步骤操作：

1. 首先，安装 pdf.js：

```bash
npm install pdfjs-dist
```

2. 在 Vue 组件中引入 pdf.js：

```javascript
import * as pdfjsLib from 'pdfjs-dist';
```

3. 创建一个 Vue 组件，用于显示 PDF 文件：

```html
<template>
  <div>
    <iframe ref="pdfIframe" @load="onLoad"></iframe>
  </div>
</template>

<script>
import * as pdfjsLib from 'pdfjs-dist';

export default {
  data() {
    return {
      pdfDoc: null,
      pageNum: 1,
      pageRendering: false,
      pageNumPending: null,
      scale: 1,
      iframe: null,
      ctx: null,
    };
  },
  mounted() {
    this.iframe = this.$refs.pdfIframe;
    this.ctx = this.iframe.contentWindow.document.body.getContext('2d');
    this.loadPdf('path/to/your/pdf/file.pdf');
  },
  methods: {
    loadPdf(url) {
      pdfjsLib.getDocument(url).promise.then((pdfDoc_) => {
        this.pdfDoc = pdfDoc_;
        this.renderPage(this.pageNum);
      });
    },
    renderPage(num) {
      this.pageRendering = true;
      this.pdfDoc.getPage(num).then((page) => {
        const viewport = page.getViewport({ scale: this.scale });
        this.iframe.height = viewport.height;
        this.iframe.width = viewport.width;

        const renderContext = {
          canvasContext: this.ctx,
          viewport: viewport,
        };
        const renderTask = page.render(renderContext);

        renderTask.promise.then(() => {
          this.pageRendering = false;
          if (this.pageNumPending !== null) {
            this.renderPage(this.pageNumPending);
            this.pageNumPending = null;
          }
        });
      });
    },
  },
};
</script>
```

4. 在上述代码中，替换 `'path/to/your/pdf/file.pdf'` 为你要显示的 PDF 文件的路径。

5. 现在，你可以在其他 Vue 组件中使用这个自定义组件来显示 PDF 文件。例如，在 App.vue 中：

```html
<template>
  <div id="app">
    <PDFViewer />
  </div>
</template>

<script>
import PDFViewer from './components/PDFViewer.vue';

export default {
  components: {
    PDFViewer,
  },
};
</script>
```

这样，你就可以在 Vue 中使用 pdf.js 渲染 PDF 文件了。




要在 Vue 中使用 PDF.js 渲染 PDF 文件并实现文本高亮，你可以按照以下步骤操作：

1. 首先，安装 pdf.js：

```bash
npm install pdfjs-dist
```

2. 在 Vue 组件中引入 pdf.js：

```javascript
import * as pdfjsLib from 'pdfjs-dist';
```

3. 创建一个 Vue 组件，用于显示 PDF 文件：

```html
<template>
  <div>
    <iframe ref="pdfIframe" @load="onLoad"></iframe>
  </div>
</template>

<script>
import * as pdfjsLib from 'pdfjs-dist';

export default {
  data() {
    return {
      pdfDoc: null,
      pageNum: 1,
      pageRendering: false,
      pageNumPending: null,
      scale: 1,
      iframe: null,
      ctx: null,
      highlightColor: 'yellow', // 设置高亮颜色
    };
  },
  mounted() {
    this.iframe = this.$refs.pdfIframe;
    this.ctx = this.iframe.contentWindow.document.body.getContext('2d');
    this.loadPdf('path/to/your/pdf/file.pdf');
  },
  methods: {
    loadPdf(url) {
      pdfjsLib.getDocument(url).promise.then((pdfDoc_) => {
        this.pdfDoc = pdfDoc_;
        this.renderPage(this.pageNum);
      });
    },
    renderPage(num) {
      this.pageRendering = true;
      this.pdfDoc.getPage(num).then((page) => {
        const viewport = page.getViewport({ scale: this.scale });
        this.iframe.height = viewport.height;
        this.iframe.width = viewport.width;

        const renderContext = {
          canvasContext: this.ctx,
          viewport: viewport,
        };
        const renderTask = page.render(renderContext);

        renderTask.promise.then(() => {
          this.pageRendering = false;
          if (this.pageNumPending !== null) {
            this.renderPage(this.pageNumPending);
            this.pageNumPending = null;
          }
        });
      });
    },
    highlightText(x, y, text) {
      this.ctx.save();
      this.ctx.fillStyle = this.highlightColor;
      this.ctx.font = 'bold 20px Arial';
      this.ctx.fillText(text, x, y);
      this.ctx.restore();
    },
  },
};
</script>
```

4. 在上述代码中，替换 `'path/to/your/pdf/file.pdf'` 为你要显示的 PDF 文件的路径。你还可以设置 `highlightColor` 属性来自定义高亮颜色。

5. 现在，你可以在其他 Vue 组件中使用这个自定义组件来显示 PDF 文件并实现文本高亮。例如，在 App.vue 中：

```html
<template>
  <div id="app">
    <PDFViewer />
  </div>
</template>

<script>
import PDFViewer from './components/PDFViewer.vue';

export default {
  components: {
    PDFViewer,
  },
};
</script>
```

这样，你就可以在 Vue 中使用 pdf.js 渲染 PDF 文件并实现文本高亮了。