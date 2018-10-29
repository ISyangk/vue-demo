<template>
    <div id="versionManageApp">
        <div :class="['inner-layout__top-part', {'inner-layout__top-part_open': !isLeftOpen},{'inner-layout__top-part_close': isLeftOpen}]">
            <i class="el-icon-menu menu_change" @click="transformMenu"></i>
            <div class="qn-main-layout__user"
                 data-step="4"
                 data-intro="在使用过程中如有问题，可点击查看帮助文档或联系客服"
                 data-disable-interaction="1"
                 data-position="bottom">
                <el-dropdown @command="handleCommand">
                        <span class="el-dropdown-link">
                            <img  id="userPhotoId" class="qn-main-layout__user-photo" :src="appData.photoUrl"  @error="setDefaultImg"/>
                            <!--<span class="text-muted text-xs block username"><b class="caret"></b></span>-->
                        </span>
                    <el-dropdown-menu slot="dropdown" class="qn-main-layout__user-dropdown-menu">
                        <el-dropdown-item command="1">修改密码</el-dropdown-item>
                        <el-dropdown-item command="2">退出</el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </div>
        </div>


        <!--侧边栏-->
        <left-menu :top-title="title" :menus="menus" > </left-menu>

        <!--右侧路由部分-->
        <div :class="['inner-layout__right-part', {'inner-layout__right-part_open': !isLeftOpen},{'inner-layout__right-part_close': isLeftOpen}]">
            <router-view ></router-view>
        </div>

        <!--修改密码框-->
        <edit-password ref="EditPassword"></edit-password>
    </div>
</template>

<script>
    import LeftMenu from '@/lib-components/leftmenu.vue';
    import EditPassword from './components/edit-password/editPassword.vue';

    var _list = [
        {
            id: '6',
            name: '个人算法',
            parentId: '0',
            hasChildren: true,
            parentName: null,
            url: '',
            actionUrl: '',
            visible: '1',
            check: false,
            children: [
                {
                    id: '1',
                    name: '用户管理',
                    parentId: '6',
                    hasChildren: false,
                    parentName: "个人算法",
                    text: "用户管理",
                    url: "/alm/user/list",
                    actionUrl: '/user/list',
                    children: []
                },
                {
                    id: '2',
                    name: '策略设置',
                    parentId: '6',
                    hasChildren: false,
                    parentName: "个人算法",
                    text: "策略设置",
                    url: "/alm/strategy/list",
                    actionUrl: '/strategy/list',
                    children: []
                }
          ]
        },
        {
            id: '8',
            name: '个人方案',
            parentId: '0',
            hasChildren: true,
            parentName: null,
            url: '',
            actionUrl: '',
            visible: '1',
            check: false,
            children: [
                {
                    id: '3',
                    name: '方案管理',
                    parentId: '8',
                    hasChildren: false,
                    parentName: "个人方案",
                    text: "方案管理",
                    url: "/alm/programme/list",
                    actionUrl: '/programme/list',
                    children: []
                },
                {
                    id: '4',
                    name: '数据统计',
                    parentId: '8',
                    hasChildren: false,
                    parentName: "个人方案",
                    text: "数据统计",
                    url: "/alm/datas/list",
                    actionUrl: '/datas/list',
                    children: []
                }
            ]
        }
    ]

    let getCheckMapTree = function(tree){
        const ignoreUrl = ["/schedule/list"];
        tree.forEach((l,i) => {
            if(l.children && l.children.length){
                l.children = getCheckMapTree(l.children);
            }
            if(~ignoreUrl.indexOf(l.actionUrl)){
                tree.splice(i,1);
            }
        });
        return tree;
    };

    export default {
        name: 'VersionManageApp',
        created() {
            // 设置用户信息
            this.$store.commit('SET_USER_INFO', JSON.parse(sessionStorage.getItem('user')));
            this.queryMenus();
        },
        data() {
            return {
                /*isCollapse: false,*/
                title: '个人网站',
                isLeftOpen: true,
                menus:[],
                appData: {
                    photoUrl: '',
                },
                // 右侧部分顶部标题名
                rightTopTitle: '',
                // 右侧路径
                fullPath:''
            }
        },
        mounted(){
        },

        computed:{


        },
        methods: {
            setDefaultImg(){
                this.appData.photoUrl = require("@/static/customer.png");
            },
            handleCommand(command){
                if(command === "1"){
                    //打开修改密码弹框
                    this.openEditPwdDialog();
                }
                else if(command === "2"){
                    //退出登录
                    this.logout();
                }
            },

            logout(){
                const loading = this.$loading({
                    lock: true,
                    text: '请稍候，正在退出xiaoying',
                    spinner: 'el-icon-loading',
                    background: 'rgba(0, 0, 0, 0.7)'
                });
                loading.close();
                window.location.href = './index.html';
            },

            transformMenu(){
                if(this.$store.state.isCollapse === true){
                    this.$store.commit('SET_IS_COLLAPSE', false);
                    this.isLeftOpen = true;

                }else{
                    this.$store.commit('SET_IS_COLLAPSE', true);
                    this.isLeftOpen = false;
                }
            },

            //查询菜单方法
            queryMenus(){
                let sessionKey = this.$store.state.userInfo.sessionKey;
                this.menus = _list;
                /*this._queryMenusTree({sessionKey}).then(rows => {
                    this.menus = rows;
                })*/
            },

            // 查询菜单(请求)
            _queryMenusTree(params) {
                return this.axios.get('/permission/queryMenuTree',{
                    params
                }).then(res => {
                    const data = res.data;
                    if(_.isString(data) && /该用户已在其他地方登录或登录会话过期,请重新登陆/i.test(data)){
                        this.$message({
                            message: "该用户已在其他地方登录或登录会话过期,请重新登陆",
                            type:"error",
                            duration:0,
                            showClose:true,
                            onClose: () => {
                            // let pathname = window.location.pathname;
                            // let re = /^\/(\w*?)\/html/i.exec(pathname);
                            // if(re.length && re[1]){
                            //     window.location.href = '/' + re[1] + '/html/login.html';
                            // }
                            // else{
                            //     window.location.href = '/html/login.html';
                            // }
                                window.location.href = './index.html';
                            }
                        });
                        return;
                    }
                    if(!data.rows){
                        return;
                    }
                    return getCheckMapTree(data.rows);
                });
            },

            /**
             * 打开修改密码框
             */
            openEditPwdDialog(){
                this.$refs.EditPassword.dialogVisible = true;
            }
        },

        components: {
            LeftMenu,
            EditPassword
        }
    }
</script>

<style lang="less">
    @import "~@/static/css/layout.css";
    @import "../less/lib-color-defined.less";
    @import "~@/less/introjs/introjs-cover";
</style>
<style>
    html{
        background-color: #eee;
        font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
    }
    body {
        margin: 0;
    }

    .qn-main-layout__user-photo{
        display:inline-block;
        height:40px;
        width:40px;
        border-radius: 20px;
    }
    .qn-main-layout__user{
        display:inline-block;
        height:40px;
        vertical-align: top;
        margin: 10px;
    }

    .qn-main-layout__user-dropdown-menu{
        min-width:100px;
    }

    .qn-main-layout__user-photo{
        display:inline-block;
        height:40px;
        width:40px;
        border-radius: 20px;
    }

    .inner-layout__top-part{
        position: fixed;
        left: 300px;
        right: 0;
        width:auto;
        height:56px;
        line-height:56px;
        background-color: #ffffff;
    }

    .qn-main-layout__user{
        float:right;
    }

    .inner-layout__right-part_open, .inner-layout__top-part_open{
        left: 64px;
        -webkit-transition: 0.6s;
        -moz-transition: 0.6s;
        -ms-transition: 0.6s;
        -o-transition: 0.6s;
        transition: 0.6s;
        /* width ease-in-out, 0.6s padding-left ease-in-out, 0.6s padding-right ease-in-out*/
    }

    .inner-layout__right-part {
        top: 56px;
    }

    .inner-layout__right-part_close, .inner-layout__top-part_close{
        left: 300px;
        -webkit-transition: 0.3s;
        -moz-transition: 0.3s;
        -ms-transition: 0.3s;
        -o-transition: 0.3s;
        transition: 0.3s;
    }

    .menu_change{
        width:40px;
        height:auto;
        margin-left:10px;
    }

    #versionManageApp{
    }

    .caret{
        display: inline-block;
        width: 0;
        height: 0;
        margin-left: 2px;
        vertical-align: middle;
        border-top: 4px dashed;
        border-right: 4px solid transparent;
        border-left: 4px solid transparent;
        margin-top:-30px;
    }

    .text-muted{
        color: #777;
    }

    .el-dropdown-link img{
        margin-right:5px;
    }

</style>
