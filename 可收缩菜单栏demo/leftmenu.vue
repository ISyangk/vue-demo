<template>
    <div class="left-sidebar__root">
        <el-menu
                class="el-menu-vertical-demo"
                :unique-opened="true"
                :router="true"
                @open="handleOpen"
                @close="handleClose"
                background-color="#324157"
                text-color="#fff"
                active-text-color="#2bb8ed"
                :default-active="defaultActive"
                :collapse="this.$store.state.isCollapse">
            <el-menu-item index="8">
                <i class="el-icon-phone-outline"></i>
                <span>{{topTitle}}</span>
            </el-menu-item>
            <template v-for="item in menus" :index="item.id">
                <template v-if="item.hasChildren" >
                    <el-submenu  :index="item.id">
                        <template slot="title" >
                            <i class="el-icon-menu"></i>
                            <span class="title-instance-left">{{item.name}}</span>
                        </template>
                        <el-menu-item v-for="subItem in item.children" :key="subItem.id" :index="subItem.actionUrl" @click="openUrl(subItem.actionUrl)">
                            <span class="title-instance-left">{{ subItem.name }}</span>
                        </el-menu-item>
                    </el-submenu>
                </template>
                <template v-else>
                    <el-menu-item :index="item.actionUrl" @click="openUrl(item.actionUrl)">
                        <label>{{ item.name }}</label>
                    </el-menu-item>
                </template>
            </template>
        </el-menu>
    </div>
</template>
<script type="text/ecmascript-6">
    module.exports = {
        name: 'Sidebar',
        props: {
            topTitle: {
                default: ''
            },
            menus: {
                type: Array
            },
        },
        created(){
            console.log(this.menus);
            this.$router.onReady(() => {
                this.defaultActive = this.$route.path;
            });
        },
        data: function(){
            return {
                defaultActive: ""
            }
        },

        methods: {
            handleOpen(key, keyPath) {
                console.log(key, keyPath);
            },
            handleClose(key, keyPath) {
                console.log(key, keyPath);
            },
            openUrl(url){
               console.log(url);
            }
        },

        watch:{
            topTitle:{
                handler:function(val){
                    this.topTitle = val;
                }
            }
        }

    };
</script>

<style lang="less" scoped>
    @import "~@/main/less/left-sidebar.less";
    .left-sidebar__list-item {
        border-left: 3px solid #fff;
        background: #fff;
    }
    .router-link-active {
        border-left: 3px solid #21d376;
        background-color: #eff4f5;
    }

    .el-menu-vertical-demo:not(.el-menu--collapse) {
        height: 100%;
        width: 300px;
    }

    .left-sidebar__root{
        height:100%;
        width:auto!important;
        position: fixed;
        top: 0;
        bottom: 0;
        left: 0;
    }

    .el-menu--collapse{
        width:64px;
        height:100%;
    }

    .title-instance-left{
        margin-left:12px;
    }
    
</style>
<style lang="less">
    .el-icon-menu+.title-instance-left{
        margin-left:0;
    }
</style>
