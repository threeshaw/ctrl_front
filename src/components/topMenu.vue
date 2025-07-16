<template>
    <div id="menu">
            <button @click.left="showlist1">文件</button>
            <li @click.left="showabout">关于</li>

        <div id="menu_file_pop" v-show="shou1 == true">
            <button @click.left="importBillExcel(row)">导入</button>
            <p>打开</p>
            <p>关闭</p>
        </div>

        <div>
            <p>版本: 1.0</p>
            <p>确认</p>
        </div>
<svg height="200" width="500">
  <polyline points="20,20 40,25 60,40 80,120 120,140 200,180" style="fill:none;stroke:black;stroke-width:3" />

</svg>


    </div>
</template>
<script setup>
import { ref } from 'vue'

    let shou1 = ref(true)

function showlist1(){
            shou1.value = !shou1.value
    }
 
            // 导入客户账单Excel表
const importBillExcel = (row) => {
    let input = document.createElement('input')
    input.type = 'file'
    input.accept = '.txt, .csv'; // 限制选择的文件类型为 .pdf, .png, .zip, .jpg
    input.style.display = 'none'
    document.body.appendChild(input)
    input.click();
    input.onchange = (e) => {
        const file = e.target.files[0] // 获取文件对象
        let paramsFile = new FormData() // 转为表单格式
        paramsFile.append('file',file) // 添加属性和值
        paramsFile.append('receivableBillSn',row.receivableBillSn) // 添加属性和值
        importCustomerBill(paramsFile).then(res=>{
            document.body.removeChild(input)
        })
    }
}



</script>
<style>
   
</style>