<template>
    <el-dialog title="添加筛选条件" :visible.sync="dialogFormVisible">
        <div class="dialog-topButton">
            <el-button size="small" type="primary"  @click="addMore">添加一条</el-button>
        </div>
        <div v-for="(list,i) in form.formList" :key="list.i">
            <el-form :model="list" :ref="'listRef' + i" >
                <el-row>
                    <el-col :span="8">
                        <el-form-item
                                label="拨打号码类型"
                                :label-width="formLabelWidth"
                                prop="callnumType"
                                :rules="[{ required: true, message: '请输入拨打号码类型', trigger: 'blur'}]">
                            <el-select v-model="list.callnumType" size="small">
                                <!--<el-option :key="phone.fieldInfo" v-for="phone in phoneTypeList" :label="phone.fieldName" :value="phone.fieldInfo"></el-option>-->
                                <el-option label="联系号码" value="field02"></el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row>
                    <el-col :span="8">
                        <el-form-item
                                label="单日拨打次数"
                                :label-width="formLabelWidth"
                                prop="numbers"
                                :rules="[{ required: true, message: '请输入单日拨打次数', trigger: 'blur'}]">
                            <el-input size="small" v-model.number="list.numbers" ></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="8">
                        <el-form-item
                                label="总拨打次数"
                                :label-width="formLabelWidth"
                                prop="sumNumber"
                                :rules="[{ required: true, message: '请输入总拨打次数', trigger: 'blur'}]">
                            <el-input size="small" v-model.number="list.sumNumber" ></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row v-for="(item,index) in list.inputTime" :key="item.key">
                    <el-col :span="10">
                        <el-form-item
                                label="间隔时间"
                                :label-width="formLabelWidth"
                                :prop="'inputTime.' + index + '.hour'"
                                :rules="inputTimeRules.hour">
                            <el-input class="inline-input" v-model.number="item.hour" size="small">
                                <span slot="suffix">小时</span>
                            </el-input>
                        </el-form-item>
                    </el-col>

                    <el-col :span="10">
                        <el-form-item
                                :label-width="formLabelWidth"
                                :key="item.key"
                                :prop="'inputTime.' + index + '.minute'"
                                :rules="inputTimeRules.minute">
                            <el-input class="inline-input" v-model.number="item.minute" size="small" >
                                <span slot="suffix">分钟</span>
                            </el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="4" class="icon_position">
                        <i class="el-icon-circle-close-outline" @click="deleteTimer(i,index)"></i>
                    </el-col>
                </el-row>
                <el-row>
                    <el-button size="small" type="primary" class="add_time_button" @click="addTimer(i)">添加间隔时间</el-button>
                </el-row>
            </el-form>
        </div>

        <div slot="footer" class="dialog-footer">
            <el-button size="small" @click="dialogFormVisible = false">取 消</el-button>
            <el-button size="small" type="primary" @click="doSubmit">确 定</el-button>
        </div>
    </el-dialog>
</template>

<script>
    import qs from "querystring";
    import service from "@/main/components/strategy/service.js";
    var _self;

    export default {
        name: 'add-one-strategy-dialog',
        props: {
            filterId: {
                required: true,
            }
        },
        data(){
            let checkHour = (rule, value, callback) =>{
                if(!value){
                    callback(new Error('时间间隔小时不能为空'));
                }else if(!Number.isInteger(value)){
                    callback(new Error('请输入数值'));
                }else {
                    callback();
                }
            }

            let checkMinute = (rule, value, callback) =>{
                if(!value){
                    callback(new Error('时间间隔分钟不能为空'));
                }else if(!Number.isInteger(value)){
                    callback(new Error('请输入数值'));
                }else {
                    callback();
                }
            }

            return {
                dialogFormVisible: false,
                formLabelWidth: "120px",
                phoneTypeList:[],
                title: '',
                operateType: '',

                form:{
                    filterId: '',
                    entId: '',
                    callnumType: [],
                    numbers: [],
                    sumNumber: [],
                    intervalTime: [],
                    hour: '',
                    minute: '',
                    strategy: '30',
                    schedule: '0',
                    period: '00:00-23:59',
                    callresultStrategy: '',
                    rounds: '0',
                    formList: [{
                        callnumType: '',
                        numbers: '',
                        sumNumber: '',
                        inputTime:[],
                        hour: '1',
                        minute: '10',
                    }],
                },
                inputTimeRules: {
                    hour: [
                        {required: true, validator: checkHour, trigger: 'blur' },
                    ],
                    minute: [
                        {required: true, validator: checkMinute, trigger: 'blur' },
                    ],
                },


            }
        },
        created() {
            _self = this ;
            this.form.formList[0].numbers = 3;
            this.form.formList[0].sumNumber = 9;
            this.form.formList[0].callnumType = 'field02';
            this.form.formList[0].inputTime.push({hour: 1, minute: 10});
        },
        methods: {
            doSubmit(){
                let newArr = [];
                let _self = this;
                this.form.formList.forEach((item,index) =>{
                   checkForm('listRef' + index);
                });

                /**
                 * 根据form表单的ref，动态生成promise，再对其做表单校验，均通过后再处理数据
                 */
                function checkForm(arrName){
                    var result = new Promise(function(resolve, reject){
                        _self.$refs[arrName][0].validate((valid) =>{
                            if(valid){
                                resolve();
                            }else{
                                reject();
                            }
                        })
                    })
                    newArr.push(result);
                };

                Promise.all(newArr).then(function() {
                    setTimeout(function(){
                        _self.addForm();
                    },1000);
                }).catch(function(){
                    _self.$message({
                        message: "操作失败",
                        type: "error"
                    });
                });
            },

            /**
             * 表单校验完成后执行新增代码
             */
            addForm(){
                const URL_MAP = {
                    'edit': '/strategy/update',
                    'add': '/strategy/add'
                };
                let _self = this;
                let url = URL_MAP[this.operateType];
                let length = _self.form.formList.length;
                _self.form.hour = _self.form.formList[length-1].inputTime[_self.form.formList[length-1].inputTime.length-1].hour;
                _self.form.minute = _self.form.formList[length-1].inputTime[_self.form.formList[length-1].inputTime.length-1].minute;
                for(var i = 0;i < length;i++){
                    _self.form.callnumType.push(_self.form.formList[i].callnumType);
                    _self.form.numbers.push(_self.form.formList[i].numbers);
                    _self.form.sumNumber.push(_self.form.formList[i].sumNumber);

                    for(var j = 0;j < _self.form.formList[i].inputTime.length; j++){
                        let times = parseInt(parseInt(_self.form.formList[i].inputTime[j].hour) * 60) + parseInt(_self.form.formList[i].inputTime[j].minute);
                        _self.form.intervalTime.push(times);

                    }
                }

                _self.form.callnumType = _self.form.callnumType.join(',');
                _self.form.numbers = _self.form.numbers.join(',');
                _self.form.sumNumber = _self.form.sumNumber.join(',');
                _self.form.intervalTime = _self.form.intervalTime.join(',');
                _self.form.entId = this.$store.state.userInfo.entId;
                _self.form.filterId = this.filterId;

                let req = Object.assign({}, this.form);

                this._addStrategy(url,req);
            },


            /**
             * 新增策略(请求)
             */
            _addStrategy(url,req){
                this.axios.post(url, qs.stringify(req)).then(resp => {
                    if (resp.data.success) {
                        this.$message({
                            message: "操作成功",
                            type: "success"
                        });
                        //this.$refs.formRef.resetFields();
                        this.$emit('refreshUneffectTable','');
                        this.form.formList.length = 1;
                        this.form.formList[0].inputTime.length = 1;
                        this.form.formList[0].callnumType = '';
                        //将数据类型更改回来
                        this.form.callnumType = [];
                        this.form.numbers = [];
                        this.form.sumNumber = [];
                        this.form.intervalTime = [];
                        this.dialogFormVisible = false;
                    }
                    else {
                        this.$message({
                            message: "操作失败",
                            type: "error"
                        });
                    }
                });
            },

            openDialog(selection){
                this.dialogFormVisible = true;
                if(selection){
                    this.form.id = selection.id;
                    this.form.formList[0].hour = Math.floor(parseInt(selection.intervalTime)/60);
                    this.form.formList[0].minute = selection.intervalTime - this.form.formList[0].hour * 60;
                    this.form.formList[0].callnumType = selection.callnumType;
                }else{
                    this.form.formList.length = 1;
                }
                service.getAllPhoneType.call(this).then(resp => {
                    if(!resp.data.success){
                        return this.$message.error("初始化下拉框失败");
                    }
                    //let data = resp.data.rows;
                    //this.phoneTypeList = data[0];
                    this.phoneTypeList = resp.data.rows;
                });
            },

            /**
             * 修改编辑或新增的标题
             */
            changeDialogType(type){
                if(type == 'add'){
                    this.title = '添加预约策略';
                    this.operateType = 'add';
                }else{
                    this.title = '编辑预约策略';
                    this.operateType = 'edit';
                }
            },

            /**
             * 添加一条策略
             */
            addMore(){
                this.form.formList.push({
                    callnumType: 'field02',
                    numbers: 3,
                    sumNumber: 9,
                    inputTime:[{hour: 1,minute: 10}],
                    hour: 1,
                    minute: 10,
                });
            },

            /**
             * 添加间隔时间(根据index)
             */
            addTimer(index){
                this.form.formList[index].inputTime.push({hour: '',minute: ''});
            },

            /**
             * 删除间隔时间(根据index)
             */
            deleteTimer(i,index){
                this.form.formList[i].inputTime.splice(index,1);
            },
        },
        computed: {

        },
        watch: {

        }
    };
</script>

<style scoped>
    .add_strategy_button{
        position: relative;
        top: -10px;
        right: -840px;
    }
    .icon_position i{
        line-height: 40px;
        width: 15px;
        height: auto;
        margin-left:30px;
    }

    .inline-input{
        width: 84%!important;
        float:left;
    }

    .inline-input + span{
        margin-left: 10px;
    }

    .dialog-topButton{

        text-align: right;
        box-sizing: border-box;
        z-index: 2;
    }

    .add_time_button{
        float: right;
    }
</style>
