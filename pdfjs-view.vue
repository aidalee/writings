<!--
 * @Author: please
 * @Date: 2023-09-27 08:31:08
 * @LastEditors: please
 * @LastEditTime: 2023-09-28 16:45:52
 * @Description: 请填写简介
-->
<script setup>
import HelloWorld from './components/HelloWorld.vue'
import * as pdfjsLib from 'pdfjs-dist/legacy/build/pdf.js'
import { PDFViewer, EventBus, PDFViewerApplication } from 'pdfjs-dist/web/pdf_viewer';
import * as PdfWorker from 'pdfjs-dist/build/pdf.worker.js'

import pdfUrl from './assets/test.pdf'
import { ref, onMounted, nextTick } from 'vue'
import { Base64 } from 'js-base64';
const iframeRef = ref(null)
let pdfDoc = ref(null)
let containerRef = ref(null)
onMounted(()=>{
  const container = containerRef.value;
  if (container == null) return;  
  // 监听事件，必须传参 PDFViewer 为实例
  const eventBus = new EventBus(null);
  eventBus.on('pagesinit', () => {
    console.log('pagesinit');
  });
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  eventBus.on('pagesloaded', (e) => {
    console.log('pagesloaded');
    console.log(e);
    // setNumPages(e.pagesCount);
  });
  eventBus.on('pagerendered', () => {
    console.log('pagerendered');
  });
  // 创建 PDFViewer
  const pdfViewer = new PDFViewer({
    container,
    eventBus,
    linkService: null,
    renderer: 'canvas',
    l10n: null,
  });

  // 导入 Document
  (async () => {
    window.pdfjsWorker = PdfWorker
    pdfjsLib.GlobalWorkerOptions.workerSrc = PdfWorker
    const loadingTask = pdfjsLib.getDocument(pdfUrl);
    const pdf = await loadingTask.promise;
    pdfViewer.setDocument(pdf);
    highlightText(container)
  })();

})
const onPdfLoad = async () => {
  window.pdfjsWorker = PdfWorker
  pdfjsLib.GlobalWorkerOptions.workerSrc = PdfWorker
  const loadingTask = pdfjsLib.getDocument(pdfUrl);
  const iframeWindow = iframeRef.value.contentWindow
  let container = iframeWindow.document.body
  loadingTask.promise.then(async pdf => {
    const totalPages = pdf.numPages;
    const pageNumber = 2;
    pdf.getPage(5).then(async page => {
      const scale = 1;
      const viewport = page.getViewport({ scale: scale });
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = viewport.width;
      canvas.height = viewport.height;
      const renderContext = {
        canvasContext: context,
        viewport: viewport
      };
      page.render(renderContext)
      highlightText(container);
    });
  });
}

const highlightText = async (container) => {
  const textToHiglighht = '性能测试'.toLowerCase()
  const allTextNodes = []
  function getTextNodes(node) {
    if(node.nodeType === 3) {
      allTextNodes.push(node)
    }else {
      for (const childNode of node.childNodes){
          getTextNodes(childNode)
        }
    }
  }
  setTimeout(()=>{
    getTextNodes(container)
    allTextNodes.forEach((textNode)=>{
      let text = textNode.textContent.toLowerCase()
      if(text.indexOf(textToHiglighht)>-1) {
        const highlightedText = `<span style="background-color: green">性能测试</span>`
        let newText = text.replace(
          '性能测试',
          highlightedText
        )
        if(newText!==text){
          const span = document.createElement('span')
          span.innerHTML = newText
          console.log(textNode.parentNode)
          textNode.parentNode.replaceChild(span, textNode)
        }
      }
    })
    // const iframeWindow = iframeRef.value.contentWindow
    // container.PDFViewerApplication.page = 5 // 传入需要让跳转的值
    console.log(PDFViewerApplication, '122')
  }, 300)
}

</script>

<template>
<div class="viewer">
  <!-- <div>url={url}</div> -->
  <!-- <div>numPages={numPages}</div> -->
  <!-- <div ref={hrRef} /> -->
  <div ref="containerRef" class="container">
    <div class="pdfViewer"></div>
  </div>
</div>
<!-- :src="'/pdf-plugin/web/viewer.html?file='+pdfUrl" -->
  <!-- <iframe width="800px" height="900px" @load="onPdfLoad" :src="'/pdf-plugin/web/viewer.html?file='+pdfUrl" ref="iframeRef"></iframe> -->
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
.viewer {
  width: 800px;
  height: 1000px;
  position: relative;
  overflow: auto;
}
.container {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    /* bottom: 0; */
    overflow: scroll;
  }
</style>
