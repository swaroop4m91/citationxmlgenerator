<script setup lang="ts">
import { VueElement, ref } from 'vue';
import formatXml from 'xml-formatter';
import { saveAs } from 'file-saver';
const fileContent = ref('');
const xmlContent = ref('');
const textLines = ref<String[]>([]);
const textAreaPlaceHolder = "Upload the text file or paste the text to genertate xml";
const itemStartTag = "<item>";
const entryStartTag = "<entry>";
const itemCloseTag = "</item>";
const entyCloseTag = "</entry>";
const referenceStartTag = "<reference-target>";
const referenceEndTag = "</reference-target>";
const updateXmlContent = () => {
    let tempXml = ''
    textLines.value=fileContent.value.split('\n')
    textLines.value.forEach((fileLine) => {
        const line = fileLine.trim();
        const index = line.lastIndexOf('-');
        const entryValue = line.substring(0, index);
        const referenceValue = line.substring(index + 1, line.length);
        tempXml += `${itemStartTag}${entryStartTag}${entryValue}${entyCloseTag}${referenceStartTag}${referenceValue}${referenceEndTag}${itemCloseTag}`;
    })
    tempXml = '<xml>' + tempXml + "</xml>"
    const formatedXml = formatXml(tempXml, {
        indentation: '   ',
        collapseContent: true
    })
    xmlContent.value = formatedXml;
}
const downloadXmlContent = ()=>{
    if (xmlContent) {
        const file = new Blob([xmlContent.value], { type: 'text/plain' });
        saveAs(file, 'generatedxml.txt')

    }
}
const initializeFileContent = () => {
    const fileInput = document.createElement('input');
    fileInput.type = 'file';
    fileInput.style.display = 'none';
    fileInput.classList.add('citation-txt-upload');
    textLines.value = [];
    fileContent.value = '';
    xmlContent.value = '';
    fileInput.onchange = (event: any) => {
        const file = event.target.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = (e: any) => {
                const textValue = e.target.result;
                fileContent.value=textValue;
                updateXmlContent();
            };
            reader.readAsText(file);
        }
    }
    document.body.appendChild(fileInput);
    fileInput.click();
}

</script>

<template>
    <div class="citation-view-box">
        <div style="width: 50%;white-space: pre-line;">
            <div style="text-align: center;"><h2>File Content</h2></div>
            <textarea v-model="fileContent" rows="50" cols="120" :placeholder="textAreaPlaceHolder" @change="updateXmlContent"></textarea>
            <div> 
                <button @click="initializeFileContent">Upload File</button>
            </div>
        </div>
        <div>
            <div style="text-align: center;"><h2>Xml Content</h2></div>
            <textarea v-model="xmlContent" rows="50" cols="120"></textarea>
            <button @click="downloadXmlContent">Download</button>
        </div>
    </div>
</template>

<style scoped>
    .citation-view-box {
        display: flex;
    }
</style>
