<template>
  <div :style="{backgroundImage:'url('+imgsrc+')',backgroundRepeat:'no-repeat',backgroundSize:'cover',height:'100%',backgroundPosition:'center'}">
      
    <div class="time-box"></div>
    <!-- <div class="time-box">第 <span>{{lotteryTime?lotteryTime:1}}</span> 轮</div> -->
    <div class="main">
        <h1 style="color:#1890ff" v-for="item in winner" :key="item.phone">{{item.name}}({{item.phone | phoneFilter}})</h1>
        <template>
            <h1 v-for="item in otherSeat" :key="item.phone">{{item.name}}({{item.phone | phoneFilter}})</h1>
        </template>
    </div>
    <button class="btn" v-show="!canStop" @click="start">开始</button>
    <button class="btn" v-show="canStop" @click="stop">停止</button>
    <!-- <div v-if="allWinner.length>0" class="winner-box">
        历史中奖: <div v-for="time in lotteryTime" :key="time">第{{time}}轮:<span v-for="(item,index) in allWinner" :key="item+index"><span v-if="item[1]==time">{{item[0]}} </span></span></div>
    </div> -->
    
    <div class="menu-box" @click="isShowSet=true">
        <img src="../../public/img/menu.png" alt="">
    </div>
    
    <div class="mask" v-show="isShowSet"></div>
    <div :style="{left:isShowSet?'0':'-30%',paddingTop:'20px'}" class="set-container">
        <div @click="isShowSet = false" class="close-btn"><img src="../../public/img/left.png" alt=""></div>
        <div class="set-box" href="javascript:void(0);" @change="addImage">
            <span>上传图片：</span><input ref="imgbg" type="file" class="fileopen" accept="image/*" />
        </div>
        <img :src="imgsrc" alt class="imgview" accept="image/*" />
        <div class="set-box">
            上传表格：<input ref="inputer" id="upload" type="file" @change="importfxx()" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel" />
        </div>
        
        <div class="set-box">共有<span style="color:red">{{nameList.length}}</span>人参与抽奖</div>
        <div class="set-box">抽奖人数：<input @blur="getInt()" type="text" v-model="winnerNum"></div>
        <div class="set-box">一次抽完：<input @click="toStopAll(true)" :disabled="intervaling" :checked="isStopAll" :value="isStopAll" type="radio" name="isOrNot">是 <input @click="toStopAll(false)" :disabled="intervaling" type="radio" :checked="!isStopAll" :value="!isStopAll" name="isOrNot">否</div>
        <div class="set-box">可重复中奖：<input @click="canRepeat(true)" :disabled="intervaling" :checked="isRepeat" :value="isRepeat" type="radio" name="repeatOrNot">是 <input @click="canRepeat(false)" :disabled="intervaling" type="radio" :checked="!isRepeat" :value="!isRepeat" name="repeatOrNot">否</div>
        

        <div class="set-btn-box">
            <button class="set-btn" @click="confirm">确认</button>
            <button class="set-btn" @click="initAll">重置</button>
            <button class="set-btn export-btn" @click="exportExcel">导出名单</button>
        </div>
        <p style="color:red;font-size:10px" v-show="showError">请上传参与抽奖人员姓名表格</p>
    </div>

    
    <!-- <div class="dio">恭喜{{sb}}中奖</div> -->
  </div>

</template>

<script>
// import data from '../../public/list.json';
import {export_json_to_excel} from '@/utils/Export2Excel';
export default {
    data(){
        return {
            isShowSet:true,//显示设置
            nameList:[],
            intNameList:[],
            awardList:[], 
            winner:[], // 当前抽奖获奖人
            allWinner:[], // 历史所有获奖人
            otherSeat:[], // 剩余抽奖位置
            winnerNum:1, // 抽奖人数
            lotteryTime:0, // 抽奖轮数
            timer:'', // 计时器
            intervaling:false, //计时器正在运行
            isStopAll:false,//是否一次抽完当次
            isRepeat:false, //是否可重复中奖
            canStop:false, // 是否可停止计时器
            clickTime: 0, // 不一次抽完时，已经抽完的次数
            imgsrc:'', // 背景图
            showError:false
        }
    },
    mounted(){
        // console.log(data)
        // this.nameList = data.list;
        // this.getInt();
        
    },
    destiory(){
        console.log('删除本地数据')
        localStorage.clear();
    },
    methods:{
        confirm(){
            if(+this.nameList.length === 0){
                this.showError = true;
            }else{
                this.isShowSet = false;
                this.showError = false
            }
        },
        initAll(){ // 完全重置
            this.toClearInterval();;
            this.canStop = false;
            this.allWinner = [];
            this.nameList = [].concat(this.intNameList);
            this.lotteryTime = 0;
            this.getInt();
        },
        toStopAll(val){
            this.isStopAll = val;
            // console.log('isStopAll:',val)
        },
        canRepeat(val){
            this.isRepeat = val;
            // console.log('isRepeat:',val)
        },
        getInt(){ // 初始化
            this.clickTime = 0;
            this.winner = [];
            this.otherSeat = [];
            let nameList = [].concat(this.nameList); // 当前抽奖的人员
            this.winnerNum = Math.min(this.winnerNum,nameList.length) // 抽奖人数
            this.lotteryTime = 0;
            let num = this.winnerNum
            // for (let i = 0; i < num; i++) {
            //     let index = Math.floor(Math.random()*nameList.length);
            //     let someOne = nameList[index]
            //     this.otherSeat.push(someOne)
            //     nameList.splice(index,1)
            //     // console.log('当前抽奖的人数',nameList,this.otherSeat)
            // }
        },
        start(){
            
            if(this.nameList.length<1){
                alert('无可参与抽奖人员')
                this.toClearInterval();;
                return
            }
            if(this.clickTime<1){
                this.winner = [];
                this.canStop = true;
            }
            if(this.clickTime<this.winnerNum){
                this.lotteryTime ++
            }
            this.timer = setInterval(() => {
                this.intervaling = true;
                this.otherSeat = []
                let nameList = [].concat(this.nameList); // 当前抽奖的人员
                // let num = Math.min(this.winnerNum,nameList.length)// 抽奖人数
                let num = this.winnerNum;
                // console.log('抽奖人数:',num,'clickTime:',this.clickTime)
                if(nameList.length<1){
                    alert('无可参与抽奖人员')
                    this.toClearInterval();;
                    return
                }
                for (let i = 0; i < num; i++) {
                    // console.log('i ,this.clickTime',i , this.clickTime)
                    if(i >= this.clickTime){
                        // console.log('进来了'+i)
                        let index = Math.floor(Math.random()*nameList.length);
                        let someOne = nameList[index]
                        if(someOne){
                            this.otherSeat.push(someOne)
                        }
                        nameList.splice(index,1)
                    }
                    // console.log('当前抽奖的人数',nameList)
                }
                
            }, 80);
        },
        stop(){
            
            if(!this.canStop){
                alert('抽奖未开始')
                return
            }

            if(this.isStopAll){
                // 一次抽完
                
                this.canStop = false;
                console.log('停止')
                this.toClearInterval();
                
                this.winner = [].concat(this.otherSeat)
                this.allWinner = this.allWinner.concat(this.otherSeat.map(item=>{return [item.name,item.phone,this.lotteryTime]}));

                // alert('恭喜'+this.otherSeat.join(',')+'中奖！')

                
                this.otherSeat = [];
                this.clickTime = 0; // 抽完重置
                // this.lotteryTime ++;
                
            }else{
                // 一个一个抽
                this.clickTime++;

                this.winner.push(this.otherSeat[0])
                this.allWinner.push([this.otherSeat[0].name,this.otherSeat[0].phone,this.lotteryTime]);

                // alert('恭喜'+this.otherSeat[0]+'中奖！')

                if(this.clickTime>=this.winnerNum){
                    this.canStop = false;
                    console.log('停止')
                    this.toClearInterval();
                    this.otherSeat = [];
                    this.clickTime = 0; // 抽完重置
                    // this.lotteryTime ++; // 一次抽一
                }
            }
            
            // console.log('this.allWinner',this.allWinner)
            localStorage.setItem('all_winner_list',JSON.stringify(this.allWinner)) // 中奖数据保存到本地

            if(!this.isRepeat){
                let allWinnerNameList = this.allWinner.map(item=>{return item[1]})
                this.nameList = this.nameList.filter((item)=>{
                    return allWinnerNameList.indexOf(item.phone) == -1
                });
                console.log('allWinnerNameList',allWinnerNameList)
            }
            if(this.nameList.length<1){ // 无可抽奖人员
                this.canStop = false;
                console.log('停止')
                this.toClearInterval();
                this.otherSeat = [];
                this.clickTime = 0; // 抽完重置
            }
            // else if(this.isStopAll){
            //     this.lotteryTime ++; // 一次抽完
            // }
            
            console.log(this.nameList)
        },
        toClearInterval(){
            clearTimeout(this.timer);
            this.intervaling = false;
        },
        importfxx() {
            let _this = this;
            console.log("xxxxxxxxxxxxxxxxxxxxxxxxxxx1");
            let inputDOM = _this.$refs.inputer;
            console.log("inputDOM",inputDOM.files[0]);
            // 通过DOM取文件数据

            this.file = inputDOM.files[0];

            var rABS = false; //是否将文件读取为二进制字符串
            var f = this.file;

            var reader = new FileReader();
            //if (!FileReader.prototype.readAsBinaryString) {
            FileReader.prototype.readAsBinaryString = function(f) {
                var binary = "";
                var rABS = false; //是否将文件读取为二进制字符串
                var pt = this;
                var wb; //读取完成的数据
                var outdata;
                var reader = new FileReader();
                reader.onload = function(e) {
                var bytes = new Uint8Array(reader.result);
                var length = bytes.byteLength;
                for (var i = 0; i < length; i++) {
                    binary += String.fromCharCode(bytes[i]);
                }
                var XLSX = require("xlsx");
                if (rABS) {
                    wb = XLSX.read(btoa(fixdata(binary)), {
                    //手动转化
                    type: "base64"
                    });
                } else {
                    wb = XLSX.read(binary, {
                    type: "binary"
                    });
                }
                outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]); //outdata就是你想要的东西
                let list = outdata.map(item=>{
                    return {
                        name:item['name'] || item['姓名'] || item['名字'],
                        phone:item['phone'] || item['电话'] || item['手机号码'] || item['电话号码']
                    }
                });
                _this.intNameList = list // 初始化姓名列表
                _this.nameList = list 
                _this.getInt();
                console.log('outdata:',outdata,list)
                };
                reader.readAsArrayBuffer(f);
            };
            if (rABS) {
                reader.readAsArrayBuffer(f);
            } else {
                reader.readAsBinaryString(f);
            }
        },
        // 上传背景图
        addImage(){
            let input = this.$refs.imgbg;
            //1. 拿到fileinput里面的文件, 这个file是一个file对象， file对象不能直接展示的
            let file = input.files[0];
        
            //2. 读取文件，成功img标签可以直接使用的格式
            //FileReader类就是专门用来读文件的
            let reader = new FileReader();
            // window.console.log(file);
        
            //3. 开始读文件
            //readAsDataURL: dataurl它的本质就是图片的二进制数据， 进行base64加密后形成的一个字符串， 这个字符串可以直接作用img标签的图片资源使用
        
            reader.readAsDataURL(file);
            let _this = this;
            //4. 因为文件读取是一个耗时操作， 所以它在回调函数中，才能够拿到读取的结果
            reader.onload = function() {
                // window.console.log(reader.result);
                //直接使用读取的结果
                _this.imgsrc = reader.result;
            };
            _this.imgsrc = file;
        },
        // 导出中奖名单
        exportExcel() {          
            // const excelHeader = this.columns.map(item => item.title)

            // const excelData = this.dataSource.map(item => keys.map(i => item[i] || ''))
            

            let localData = localStorage.getItem('all_winner_list');
            if(!localData){
                alert('暂无中奖名单')
            }else{
                const excelHeader = ['姓名','电话','中奖轮数'] 
                const excelData = JSON.parse(localData)
                const excelName='中奖名单'
                export_json_to_excel(excelHeader, excelData, excelName)
            }
            

        },
    },
    filters: {
        phoneFilter: function (value) {
            if (!value) return ''
            value = value.toString()
            return value.slice(0,3)+'*****'+value.slice(-3)
        }
    }
}
</script>

<style scoped>
.main{
    width: 1050px;
    min-height: 50%;
    text-align: center;
    margin: 50px auto;
    display: flex;
    justify-content: flex-start;
    flex-wrap: wrap;
    align-items: center;
}
.main>h1{
    display: inline-block;
    width: 50%;
}
.main>h1:nth-of-type(1){
    margin: 0 auto;
}
.btn,.set-btn{
    width: 80px;
    height: 50px;
    border-radius: 4px;
    color: #fff;
    background-color: #1890ff;
    border: 1px solid #DCDFE6;
    border-color: #1890ff;
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    -webkit-transition: .1s;
    transition: .1s;
    opacity: 1;
}
.set-btn{
    width: auto;
    padding: 0 20px;
    height: 30px;
}
.btn:hover,.set-btn:hover{
    background: #4a9bfa;
    opacity: 0.8;
}
.btn:active,.set-btn:active{
    background: #0075fb;
    opacity: 0.8;
}
.set-btn-box{
    display: flex;
    justify-content: space-around;
    padding: 0 20%;
    margin-top: 50px;
}
.export-btn{
    background-color: #13ce66;
    border-color: #13ce66;
}
.export-btn:hover{
    background: #13ce66;
    opacity: 0.8;
}
.export-btn:active{
    background: #13ce66;
    opacity: 0.8;
}
button, html [type="button"], [type="reset"], [type="submit"] {
    -webkit-appearance: button;
}
.btn1{
    width: 80px;
    height: 80px;
    border-radius: 50%;
    box-shadow: none;
    border: none;
    background-image: radial-gradient(red,yellow);
    color: #FFF;
    font-weight: 500;
    font-size: 18px;
}
.mask{
    position: fixed;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    background-color: rgba(0, 0, 0, 0.404);
}
.time-box{
    letter-spacing:.1rem;
    font-size: 80px;
    padding-top: 100px;
}
.time-box span{
    font-weight: 600;
    font-size: 120px;
    color: #4a9bfa;
}
.menu-box{
    position: fixed;
    left: 10px;
    top: 10px;
}
.menu-box img{
    width: 30px;
}
.close-btn{
    text-align: right;
    padding-right: 20px;
}
.close-btn img{
    width: 40px;
}
.set-container{
    position: fixed;
    box-sizing: border-box;
    top: 0;
    left: 0;
    background-color: rgba(238, 238, 238, 0.801);
    height: 100vh;
    width: 30%;
    padding-left: 20px;
    transition: all 0.5s;
}
.set-box{
    width: 400px;
    margin: 0 auto;
    text-align: left;
    margin-top: 30px;
    font-size: 14px;
    color: #666;
}
.set-box input{
    line-height: 40px;
    border:1px solid #666;
    border-radius: 8px;
    box-shadow: none;
    text-align: center;
    width: 40px;
}
input[type="file" i] {
    width: auto;
    border: none;
}
.winner-box{
    width: 400px;
    min-height: 200px;
    margin: 0 auto;
    margin-top: 30px;
    font-size: 14px;
    color: #666;
    line-height: 30px;
    text-align: left;
}
.imgview{
    width: 40%;
}
</style>