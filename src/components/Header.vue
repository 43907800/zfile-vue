<template>
    <div>
        <el-form :inline="true" class="zfile-header" size="mini">
            <el-form-item>
                <el-breadcrumb separator="/" separator-class="el-icon-arrow-right">
                    <el-breadcrumb-item :to="{path: '/' + driveId + '/main'}">{{this.$store.state.common.config.siteName ? this.$store.state.common.config.siteName : '首页'}}</el-breadcrumb-item>
                    <el-breadcrumb-item v-for="item in breadcrumbData" class="hidden-xs-only"
                                        :to="{path: '/' + driveId + '/main' + item.fullPath}"
                                        :key="item.path">{{item.name}}</el-breadcrumb-item>
                </el-breadcrumb>
            </el-form-item>
            <div class="zfile-header-drive box animate__animated animate__fadeIn">

            <el-form-item v-show="this.$store.getters.debugMode" label="已开启 DEBUG 模式，使用完请及时关闭" class="zfile-debug-tips" size="small">
                <el-button @click="resetAdminPwd" size="small" type="danger">重置密码</el-button>
            </el-form-item>

                <span @click="recordFormVisible =true" style="margin-right: 20px" >记录</span>

                <el-form-item label="图片模式" size="small">
                    <el-switch v-model="imgModel"></el-switch>
                </el-form-item>

                <el-select size="small" v-model="currentDriveId" placeholder="请选择驱动器">
                    <el-option v-for="item in driveList"
                            :key="item.id"
                            :label="item.name"
                            :value="item.id">
                    </el-option>
                </el-select>
            </div>
        </el-form>
        
        <el-dialog title="记录查询" :visible.sync="recordFormVisible" :width="dialogWidth">
            <el-form :model="recordForm" :inline="true" class="demo-form-inline" label-position="left">
                <el-form-item label="用户名">
                    <el-input v-model="recordForm.userName" placeholder="用户名"></el-input>
                </el-form-item>
                <el-form-item label="状态">
                    <el-select v-model="recordForm.status" placeholder="活动区域">
                    <el-option label="最新" value="1"></el-option>
                    <el-option label="全部" value=""></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="allRecord()">确定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    import path from 'path'

    export default {
        name: "Header",
        props: ['driveId'],
        data() {
            return {
                imgModel: false,
                driveList: [],
                currentDriveId: "",
                search: '',
                recordFormVisible: false,
                breadcrumbData: [],
                searching: false,
                recordForm:{
                    userName: this.$cookies.get("username"),
                    filePath: this.common.getPath(),
                    status: '1'
                }
            }
        },
        created() {
            this.buildBreadcrumbData();
        },
        computed: {
            dialogWidth() {
                return document.body.clientWidth>500?'50%':'80%';
            }
        },
        methods: {
            resetAdminPwd() {
                this.$confirm('是否确认重置后台管理员密码？重置后用户名/密码将强制修改为 admin 123456', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                    callback: action => {
                        if (action === 'confirm') {
                            this.$http.get('debug/resetPwd').then((response) => {
                                if (response.data.code === 0) {
                                    this.$message.success("重置成功，请及时关闭 debug 功能，防止出现安全问题！");
                                } else {
                                    this.$message.error(response.data.msg)
                                }
                            });
                        }
                    }
                });
            },
            buildBreadcrumbData() {
                this.breadcrumbData = [];
                let fullPath = this.$route.params.pathMatch;
                fullPath = fullPath ? fullPath : '/';

                while (fullPath !== '/') {
                    let name = path.basename(fullPath);
                    this.breadcrumbData.unshift({name, fullPath});
                    fullPath = path.resolve(fullPath, "../");
                }
            },popUserName() {
                let username = this.$cookies.get("username");
                this.$prompt('请输入用户名', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    inputValidator(val) {
                        return !!val
                    },
                    inputValue: username,
                    inputErrorMessage: '用户名不能为空.'
                }).then(({value}) => {
                    this.$cookies.set("username",value,-1); 
                    this.recordForm.userName = value;
                    this.allRecord() 
                }).catch(() => {

                });
            },
            allRecord() {
                this.$cookies.set("username",this.recordForm.userName,-1); 
                this.recordForm.filePath =this.common.getPath();
                this.$http.get('record/findUserRecord',{params: this.recordForm}).then((response) => {
                    let res = response.data;
                    let _html ='<div style=" height: 200px; overflow: auto; width: 110%; ">';
                    res.data.forEach((item)=>{
                        _html+='<p>'+item.filePath+'<span style=" color: red; ">'+item.fileName+'</span>&emsp;<span style=" color: turquoise; ">'+item.createTime+'</span></p><hr/>'
                    })
                    _html +='</div>';
                    this.$notify({
                        title: '历史记录',
                        dangerouslyUseHTMLString: true,
                        // duration: 0,  // 定时关闭 0不关闭
                        message: _html
                    });

                })
            }
        },
        watch: {
            'currentDriveId': function (newVal, oldVal) {
                this.$store.commit('updateOldDriveId', oldVal);
                this.$router.push('/' + newVal + '/main');
            },
            '$store.getters.newImgMode': function (newVal) {
                this.imgModel = newVal;
            },
            'imgModel': function () {
                this.$store.commit('switchImgMode', this.imgModel);
            },
            '$route.fullPath': function () {
                // 当 URL 变化, 则自动重新 build 面包屑
                this.buildBreadcrumbData();
            },
            'search': function (newVal) {
                clearTimeout(this.timer);
                this.timer = setTimeout(() => {
                    this.$store.commit('updateSearchParam', newVal);
                }, 150);
            }
        },
        async mounted() {
            await this.$http.get('api/drive/list').then((response) => {
                this.driveList = response.data.data;
                // 如果当前 URL 参数中有驱动器 ID, 则直接用当前的.
                if (this.driveId) {
                    this.currentDriveId = Number(this.driveId);
                } else if (this.driveList.length > 0) {
                    // 否则读取驱动器列表中的第一个, 并跳转到响应的 URL 中.
                    this.currentDriveId = this.driveList[0].id;
                    this.$router.push('/' + this.driveList[0].id + '/main');
                } else if (this.driveList.length === 0) {
                    this.$message.warning( '无可用驱动器，请先去管理员页初始化驱动器。');
                }

                this.driveList.some((item) => {
                    if (item.id === this.currentDriveId) {
                        this.$store.commit('updateCurrentStorageStrategy', item);
                    }
                });
            });
        }
    }
</script>

<style scoped>
    .zfile-header {
        display: flex;
        flex-flow: row nowrap;
        justify-content: space-between;
        height: 48px;
        line-height: 48px !important;
        background: #fafafa;
        border-bottom: 1px solid rgba(0,0,0,0.05);
        padding-left: 30px;
    }

    .zfile-header .el-breadcrumb {
        line-height: 48px;
        font-size: 13px;
    }

    .zfile-header .el-input {
        line-height: 48px;
    }


    @media only screen and (max-width:767px) {
        .zfile-header >>> .el-breadcrumb__separator {
            display: none !important;
        }
        .zfile-header >>> .el-form-item__label {
            display: none !important;
        }
        .zfile-header >>> .el-select {
            width: 120px;
        }
    }

    .zfile-header-drive {
        margin-right: 20px;
    }

    .zfile-header-drive >>> .el-form-item__content {
        vertical-align: unset;
    }

    .zfile-debug-tips >>> .el-form-item__label {
      font-weight: bold;
      color: red !important;
    }

</style>
