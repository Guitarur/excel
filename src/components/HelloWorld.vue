<template>
  <div class="hello">
    <div class="copy_text">
      <span class="su" v-if="copyKey == 1">复制成功</span>
      <span class="failed" v-if="copyKey == 2">复制失败</span>
    </div>
    <input class="file_input" ref="file" type="file" accept=".xls" id="xlf" @change="change_file" />

    <div class="my_table" v-for="(item,index) in newArr" :key="index">
      <span>{{item['测站编码']}}</span>
      <span>{{item['观测时间']}}</span>
      <span>{{item['地下水埋深']}}</span>
      <span class="copy_btn" v-clipboard:copy="item['地下水埋深']" v-clipboard:success="onCopy" v-clipboard:error="onError">Copy</span>
      
    </div>
  </div>
</template>

<script>

import XLSX from 'xlsx'
export default {
  name: 'HelloWorld',
  data () {
    return {
      tableData: [],
      fileExcel: {},
      file: '',
      data: '',
      newArr: [],
      copyKey: 0
    }
  },
  methods: {
    onCopy () {
      this.copyKey = 1
      setTimeout(() => {
        this.copyKey = 0
      }, 1000);
    },
    onError () {
      this.copyKey = 2
        
    },
    change_file(e){
        if(this.$refs.file && !this.$refs.file.files[0]) return false;
        const selected_file = this.$refs.file.files[0];
        this.fileExcel.oldFile = selected_file;
        this.file_name = selected_file.name;
        this.file_size = selected_file.size ? (selected_file.size / 1024).toFixed(2) : 0;
        this.readFule(selected_file)
      },
    readFule(file){
      // console.log(file)
      var name = file.name;
      var reader= new FileReader();
      let that = this;
      reader.onload = function (e) {
        console.log(e)
          that.fileExcel.oldFile = e.target.result;
          var wb = XLSX.read(that.fileExcel.oldFile, { type: "binary",cellDates: true });
          // console.log(wb);
          
          var wb_sheetname = wb.SheetNames[0];
          const wb_sheet = wb.Sheets[wb_sheetname]
          // console.log(wb_sheet);
          let fromTo = "";
          fromTo = wb_sheet['!ref'];
          // console.log(fromTo + "fromTo")
          that.selectFileData = XLSX.utils.sheet_to_json(wb_sheet);
          console.log(that.selectFileData);
          that.init(that.selectFileData)
          if(!that.selectFileData.length){
            tipsMsg(that,'error',`上传的文件数据为空！`);
            return false
          }
          // that.get_result();
      };
      reader.readAsBinaryString(file);
    },

    init (data){
      let newArr = []
      console.log(data[0]['观测时间'].getHours())
      newArr = data.filter((ele,index) => {
        // data[index]['观测时间'].getDate()
        return (new Date(ele['观测时间']).getDate()) == 26 && (new Date(ele['观测时间']).getHours() == 4)
      });
      // this.newArr = newArr
      newArr.forEach((ele,index) => {
        newArr[index]['观测时间'] = this.initDate(ele['观测时间'])
      })
      console.log(newArr)
      this.newArr = newArr
    },
    initDate (time){
      return time.getFullYear() + '-' + time.getMonth() + '-' + time.getDate()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .file_input {
    margin: 10px;
  }
  .my_table {
    margin:  auto;
    width: 500px;
    padding: 5px 0;
  }
  .my_table:nth-child(2n){
    background-color: #e8e8e8;
  }
  .my_table span{
    display: inline-block;
    width: 100px;
  }
  .copy_text {
    position: absolute;
    top: 10px;
    left: 50%;
    margin: auto;
    
  }
  .copy_text .su{
    padding: 10px;
    background-color: rgb(172, 223, 172);
    color: green;
  }
  .copy_text .failed {
    padding: 10px;
    background-color: rgb(223, 197, 172);
    color: red;
  }
  .copy_btn {
    cursor: pointer;
    border: 1px solid black;
  }
</style>
