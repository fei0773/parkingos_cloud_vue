<template>
  <section style="padding: 100px;">

    <el-row class="align-center" style="margin-left: 22%;">
        <span style="font-size:20px">扫码全免</span>
    </el-row>
    </br></br></br>
    <el-row class="align-center" style="margin-left: 20%;">
            <div>
            <el-form :model="freeCodeReduce" label-width="80px" :rules="withdrawFormRules" ref="freeCodeReduce">

				<el-form-item  prop="reduce">
					<el-input v-model="freeCodeReduce.reduce" style="width:84%" placeholder="全免券" disabled></el-input>
					<el-button @click="getFreeTicketCode" type="primary" size ="small" style="height: 38.5px;margin-top: -2px;">获 取</el-button>
					<div v-show="showFree" style="margin-top:15px;float:left">
					    <span>
					        扫码一次：
					        <template >
					             <el-radio v-model="free_limit_times" label="0" v-on:change="changeReadonly">单次有效</el-radio>
                                 <el-radio v-model="free_limit_times" label="1" v-on:change="changeReadonly">多次有效</el-radio>
                            </template>
					    </span>
					</div>
					<div style="margin-left:-14%" v-show="showFree">
					    <el-input v-model="freeCodeReduce.freeLimit" style="width:74%" placeholder="请输入有效时间,单位小时" :readonly="readonly"></el-input>
					</div>

				</el-form-item>
		    </el-form>
       </div>
    </el-row>


     <div  class="align-center" style="margin-left: 20%;padding: 20px;">
       <el-checkbox id="box" v-model="freeCodeReduce.isauto" :disabled="checkisdisable" ></el-checkbox>
       <span>自动更新</span>
     </div>

       <el-row  class="align-center" style="margin-left: 25%;">
                    <el-col :span="23" :offset="1" align="middle" style="margin-top: 10px;margin-bottom:5px">
                        <!--二维码图片:-->
                    </el-col>
                <el-row>
                    <!--<el-col :span="10" :offset="1" >
                        <el-checkbox-group v-model="checkQrBox" @change="changeQrBox" style="margin-top:10px;margin-left:12px">
                            <el-row style="line-height:30px;display:inline"><el-checkbox label="1">平台环境</el-checkbox></el-row>
                            <el-row style="line-height:30px;display:inline"><el-checkbox label="2">车场名称</el-checkbox></el-row>
                            <el-row style="line-height:30px;display:inline"><el-checkbox label="3">二维码类型</el-checkbox></el-row>
                            <el-row v-if="needChannel" style="line-height:30px;display:inline"><el-checkbox label="4">通道编号</el-checkbox></el-row>
                        </el-checkbox-group>
                    </el-col>-->
                    <el-col :span="12" :offset="1" style="margin: 0 auto;">
                        <canvas id="freecanvas" style="display:none"></canvas>
                        <canvas id="freeimg" style="display:none"></canvas>
                        <div >
                            <img :src="freeqrsrc" style="margin-left: 45%;"/>
                        </div>

                    </el-col>
                </el-row>
            </el-row>

	</el-dialog>





  </section>

</template>

<script>
import { path,server,carditems,checkPhone,dtypelist,cardtypeitems,otypelist,accountitems,belongitems,settleitems,percision } from '../api/api';
import common from '../common/js/common'
import CommonTable from '../components/CommonTable'

export default {
  components:{
    CommonTable
  },
  data(){
    return{
      readonly:false,
      free_limit_times:'0',
      showFree:false,
      loading: false,
      infoloading: false,
      cycleisdisable:false,
      moneyisdisable:false,
      checkisdisable:false,
      withdrawFormVisible:false,
      freeCodeVisible:false,
      setupVisible:false,
      freeCarNumberVisible:false,
      addloading:false,

      codeReduce:{
        reduce:'',
        isauto:false,
      },
      freeCodeReduce:{
          reduce:'全免券',
          isauto:false,
          freeLimit:'',
      },
      carNumReduce:{
         reduce:'',
         car_number:'',
      },
      freecarNumReduce:{
         reduce:'全免券',
         car_number:'',
     },
      handInputType: [
          {'value_name': '不支持', 'value_no': '0'},
          {'value_name': '支持', 'value_no': '1'}
      ],
      withdrawLoading:false,
      ticketUnit:'元',
      type:'3',
      code:'',
      freecode:'',
      ticket_url:'',
      free_ticket_url:'',
      noStation:false,
      account:{},
      card:{},
      cardid:'',
      qrsrc:'',
      freeqrsrc:'',
      nocard:false,
      bankname:'无',
      cardnum:'暂未绑定银行卡',
      shopname:'获取中...',
      ticketLimit:'获取中...',
      ticketfree_limit:'获取中...',

      withdrawForm:{
        money:''
      },
      form:{
        name:''
      },
      status:'',

      infoModify:{
        id:'',
        hand_input_enable:'',
        validite_time:'',
        default_limit:''
      },
      accountModify:{
        id:'',
        name:'',
        address:'',
        phone:''
      },
      parkstations:[],
      temp:{},
      infotemp:{},
      accountFormRules:{
        name: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        phone: [
          { validator:checkPhone, required: true,  trigger: 'blur' }
        ],
        address: [
          { required: true, message: '请输入地址', trigger: 'blur' }
        ],
      },
      infoFormRules:{
        default_limit: [
          { required: true, message: '请输入默认显示额度', trigger: 'blur' }
        ],
        validite_time: [
          { required: true, message: '请输入有效期',  trigger: 'blur' }
        ]
      },
      withdrawFormRules:{

      },
    }
  },
  mounted(){
     let vm = this
     vm.getShopAccountInfo()
     window.setInterval(this.getCodeStatus,10000)
     window.setInterval(this.getFreeCodeStatus,10000)
  },
  methods: {
        changeReadonly(readonly){
            let vm = this;
            if(readonly==0){
                vm.freeCodeReduce.freeLimit=''
                vm.readonly=true
            }else{
                vm.readonly=false
            }
        },
     handleShowOrderDetail() {
        //跳转到订单详情
        //this.$router.push({path: '/index'});

        window.open("http://localhost:8086/#/code_reduce");

     },
    selectChecked(){
        var vm=this;
        window.setTimeout(()=> {
            if(vm.checkisdisable){

            }else{
                vm.moneyisdisable = false
                vm.cycleisdisable = false
                if(!vm.setup.checked){
                    vm.setup.money=""
                    vm.setup.cycle=""
                    vm.moneyisdisable = true
                    vm.cycleisdisable = true
                }else{
                     vm.moneyisdisable = false
                     vm.cycleisdisable = false
                }
            }
        },0)
    },
    downloadQr(){
				location=this.qrsrc
			},
    	genqr(url){

//  		console.log('111111111111111111111'+text)

				var canvas = document.getElementById('canvas')
				//console.log(canvas)
				this.QRCode.toCanvas(canvas, url,{ errorCorrectionLevel: 'H' }, function (error) {
					//console.log(url)
					if (error){
						//console.error(error)
					} else{
						//console.log('success!');
					}
				})
				//console.log(canvas.width)
				var context=canvas.getContext('2d');
           		var imageData = context.getImageData(0,0,canvas.width,canvas.height);

				var img = document.getElementById("freeimg");
				img.width=canvas.width
				img.height=canvas.height
				var context2 = img.getContext('2d');
				context2.fillStyle="white";
				context2.fillRect(0,0,canvas.width,(canvas.height));
				context2.putImageData(imageData,0,0);
				context2.font="bold 10px 微软雅黑"
				context2.fillStyle="black"

				var url = img.toDataURL("image/png");
				console.log(url+'---------------------------')
				this.qrsrc = url
			},

			freegenqr(url){

            //  		console.log('111111111111111111111'+text)

            				var canvas = document.getElementById('freecanvas')
            				//console.log(canvas)
            				this.QRCode.toCanvas(canvas, url,{ errorCorrectionLevel: 'H' }, function (error) {
            					//console.log(url)
            					if (error){
            						//console.error(error)
            					} else{
            						//console.log('success!');
            					}
            				})
            				//console.log(canvas.width)
            				var context=canvas.getContext('2d');
                       		var imageData = context.getImageData(0,0,canvas.width,canvas.height);

            				var img = document.getElementById("freeimg");
            				img.width=canvas.width
            				img.height=canvas.height
            				var context2 = img.getContext('2d');
            				context2.fillStyle="white";
            				context2.fillRect(0,0,canvas.width,(canvas.height));
            				context2.putImageData(imageData,0,0);
            				context2.font="bold 10px 微软雅黑"
            				context2.fillStyle="black"

            				var url = img.toDataURL("image/png");
            				console.log(url+'---------------------------')
            				this.freeqrsrc = url
            //				var filename = this.initunionid+"-"+this.parkid+"-"+this.randomNum(6)+".png"
            //				var download = document.getElementById("#download")
            //$axios
            //				var triggerDownload = this.download.eval("href", url).eval("download", filename);
               				//triggerDownload[0].click();
            			},
    randomNum(n){
				var t='';
				for(var i=0;i<n;i++){
				t+=Math.floor(Math.random()*10);
				}
				return t;
			} ,
    cycleinputFunc(){
     var vm=this;
     vm.moneyisdisable = false
     vm.checkisdisable = false
     if(this.setup.cycle!=null&&this.setup.cycle!=""){
        vm.setup.money=""

     }
    },
     moneyinputFunc(){
         var vm=this;
         vm.cycleisdisable = false
         vm.checkisdisable = false
         if(this.setup.money!=null&&this.setup.money!=""){
            vm.setup.cycle=""

         }
        },

    getCodeStatus(){
         let vm = this;
        vm.$axios.post(path+"/shopticket/ifchangecode?code="+vm.code,{
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
            }
        }).then(function (response) {
          let ret = response.data;
          console.log('陈博文'+ret.state);
          if(ret.state==1){
          	vm.getTicketCode();
          	vm.code='';
          	vm.$message({
                 message: "二维码已更新" ,
                 type: 'success',
                 duration: 1200
             });
             //this.$options.methods.getTicketCode();
          }
        });
    },
    getFreeCodeStatus(){
             let vm = this;
            vm.$axios.post(path+"/shopticket/ifchangecode?code="+vm.freecode,{
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                }
            }).then(function (response) {
              let ret = response.data;
              console.log('陈博文'+ret.state);
              if(ret.state==1){
              	vm.getFreeTicketCode();
              	vm.freecode='';
              	vm.$message({
                     message: "二维码已更新" ,
                     type: 'success',
                     duration: 1200
                 });
                 //this.$options.methods.getTicketCode();
              }
            });
        },
    getFreeTicketCode(){
         let vm = this;
         vm.type=4;

         vm.$refs.freeCodeReduce.validate((valid) => {
             if (valid) {
                 vm.checkisdisable=true,
                 vm.$axios.post(path+"/shopticket/createticket?shopid="+sessionStorage.getItem('shopid')+"&type="+vm.type+"&isauto="+(vm.freeCodeReduce.isauto?1:0)+"&free_limit_times="+vm.free_limit_times+"&time_range="+vm.freeCodeReduce.freeLimit,{
                     headers: {
                         'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                     }
                 }).then(function (response) {
                     let ret = response.data;
                     //var ret = eval('('+result+')')
                     if(ret.state==1){
                         vm.freecode = ret.code;
                         vm.free_ticket_url = ret.ticket_url;
        //               console.log(vm.code+"1111111111111111111111111111111")
                         vm.freegenqr(vm.free_ticket_url)
                     }else{
                         vm.$message({
                             message: "获取失败" + ret.error,
                             type: 'error',
                             duration: 1200
                         });
                     }

                 });
             }
         });


    },
    getTicketCode(){
         let vm = this;
            vm.$refs.codeReduce.validate((valid) => {
                if (valid) {
                  vm.$axios.post(path+"/shopticket/createticket?shopid="+sessionStorage.getItem('shopid')+"&type="+vm.type+"&reduce="+vm.codeReduce.reduce+"&isauto="+(vm.codeReduce.isauto?1:0),{
                                 headers: {
                                     'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                                 }
                             }).then(function (response) {
                                 let ret = response.data;
                                 //var ret = eval('('+result+')')
                                 if(ret.state==1){
                                 	vm.code = ret.code;
                                     vm.ticket_url = ret.ticket_url;
                 //               console.log(vm.code+"1111111111111111111111111111111")
                                     vm.genqr(vm.ticket_url)
                                 }else{
                                     vm.$message({
                                         message: "获取失败" + ret.error,
                                         type: 'error',
                                         duration: 1200
                                     });
                                 }

                             });
                }
            });
    },

    freeuseTicketByCarNumber(){
        let vm = this;
        vm.type = 4;
        vm.$axios.post(server+"/zld/shopticket?action=noscan&shop_id="+sessionStorage.getItem('shopid')+"&car_number="+encodeURI(encodeURI(vm.freecarNumReduce.car_number))+"&type=1",{
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
            }
        }).then(function (response) {
            let ret = response.data;
            //var ret = eval('('+result+')')
            if(ret.result==1){
                vm.$message({
                    message: '用券成功!',
                       type: 'success',
                       duration: 1200
                   });
            }else{
                vm.$message({
                    message: ret.error,
                    type: 'error',
                    duration: 1200
                });
            }

        });

    },

    useTicketByCarNumber(){
    let vm = this;
    vm.$axios.post(server+"/zld/shopticket?action=noscan&shop_id="+sessionStorage.getItem('shopid')+"&car_number="+encodeURI(encodeURI(vm.carNumReduce.car_number))+"&type="+vm.type+"&reduce="+vm.carNumReduce.reduce,{
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
        }
    }).then(function (response) {
        let ret = response.data;
        //var ret = eval('('+result+')')
        if(ret.result==1){
            vm.$message({
                message: '用券成功!',
                   type: 'success',
                   duration: 1200
               });
        }else{
            vm.$message({
                message: ret.error,
                type: 'error',
                duration: 1200
            });
        }

    });

},
    getShopAccountInfo(){
      let vm = this;
      vm.$axios.post(path+"/shopaccount/shopinfo?id="+sessionStorage.getItem('shopid'),{
          headers: {
              'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
          }
      }).then(function (response) {
        let ret = response.data;
        //var ret = eval('('+result+')')
        vm.account=ret;
        //alert(ret.free_limit_times)
        if(ret.free_limit_times==1){
            vm.showFree = true;
            vm.free_limit_times='1';
        }
        if(ret.hand_input_enable==1){
             vm.infoModify.hand_input_enable='支持';
        }else{
            vm.infoModify.hand_input_enable='不支持';
        }
        vm.shopname=ret.name
        if(ret.ticket_unit==1){
            vm.ticketLimit=ret.ticket_limit
            vm.ticketUnit = '分钟'
        }else if(ret.ticket_unit==2){
            vm.ticketLimit=ret.ticket_limit
            vm.ticketUnit = '小时'
        }
        else if(ret.ticket_unit==3){
            vm.ticketLimit=ret.ticket_limit
            vm.ticketUnit = '天'
        }else if(ret.ticket_unit==4){
             vm.ticketLimit=ret.ticket_money
             vm.ticketUnit = '元'
             vm.type = '5'
         }

        vm.ticketfree_limit=ret.ticketfree_limit
        vm.accountModify.id=ret.id
        vm.accountModify.name=ret.name
        vm.accountModify.address=ret.address
        vm.infoModify.id=ret.id
        vm.infoModify.default_limit=ret.default_limit
        vm.infoModify.validite_time=ret.validite_time
        vm.temp=common.clone(vm.accountModify)
        vm.infotemp=common.clone(vm.infoModify)
      });
    },

    editSubmit() {
        //发送ajax,提交表单更新
        let vm = this;
        var eform=this.accountModify;
        eform = common.generateForm(eform);
        vm.$refs.accountModify.validate((valid) => {
            if (valid) {
                vm.loading = true;
                vm.$axios.post(path + "/shopaccount/edit", vm.$qs.stringify(eform), {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                    }
                }).then(function (response) {
                    let ret = response.data;
                    //alert(ret.state);
                        if (ret.state == 1) {
                            //更新成功
                            vm.getShopAccountInfo();
                            vm.$message({
                                message: '更新成功!',
                                type: 'success',
                                duration: 600
                            });
                        } else {
                            //更新失败
                            vm.$message({
                                message: '更新失败!' + ret.msg,
                                type: 'error',
                                duration: 600
                            });
                        }
                         vm.loading = false;
                }).catch(function (error) {
                    setTimeout(() => {
                       // vm.alertInfo('请求失败!' + error);
                    }, 150);
                });
            }
        });
    },
       infoEditSubmit() {
        //发送ajax,提交表单更新
        let vm = this;
        var eform=this.infoModify;
        eform = common.generateForm(eform);
        vm.$refs.infoModify.validate((valid) => {
            if (valid) {
                vm.infoloading = true;
                vm.$axios.post(path + "/shopaccount/infoedit", vm.$qs.stringify(eform), {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
                    }
                }).then(function (response) {
                    let ret = response.data;
                    //alert(ret.state);
                        if (ret.state == 1) {
                            //更新成功
                            vm.getShopAccountInfo();
                            vm.$message({
                                message: '更新成功!',
                                type: 'success',
                                duration: 600
                            });
                        } else {
                            //更新失败
                            vm.$message({
                                message: '更新失败!' + ret.msg,
                                type: 'error',
                                duration: 600
                            });
                        }
                         vm.infoloading = false;
                }).catch(function (error) {
                    setTimeout(() => {
                       // vm.alertInfo('请求失败!' + error);
                    }, 150);
                });
            }
        });
    },

    resetEdit(){
      this.accountModify=common.clone(this.temp)
    },
     inforesetEdit(){
      this.infoModify=common.clone(this.infotemp)
    },
    closeWithdraw(){
      this.codeReduce.reduce=''
      this.codeReduce.isauto=false
    },
    closesetup(){
        this.carNumReduce.reduce = ''
        this.carNumReduce.car_number = ''
    },
    closeFreeCar(){
        //this.freecarNumReduce.reduce = ''
        this.freecarNumReduce.car_number = ''
    },
    closeFreeCode(){
        //this.freeCodeReduce.reduce=''
        this.freeCodeReduce.isauto=false
    }
  },
  activated(){

      //this.getAutoWithDraw()
      this.getShopAccountInfo()

      window.setInterval(this.getCodeStatus,10000)
      window.setInterval(this.getFreeCodeStatus,10000)
      //this.getParkCardInfo()
      //this.getParkStatus()
      //window.setInterval(this.getParkStatus,10000)
      //window.setInterval(this.getParkAccountInfo,120000)
	},
  watch:{
    account:function(val,oldVal){
      this.account = val
    }
  }

}

</script>

<style>
  .parkstatus{
    margin-top:5px
  }
  .statustext{
    font-weight:bold;margin-left:10px;color:#9B9EA0
  }
  .align-center{
        margin:0 auto; /* 居中 这个是必须的，，其它的属性非必须 */
        width:500px; /* 给个宽度 顶到浏览器的两边就看不出居中效果了 */

        text-align:center; /* 文字等内容居中 */
  }
</style>
