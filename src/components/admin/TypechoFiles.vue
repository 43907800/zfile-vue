<template>
    <el-card class="zfile-admin-short-card">
        <el-form :inline="true" ref="searchForm" class="zfile-admin-short-form" :model="searchParam">
<!--            <el-form-item label="标题">-->
<!--                <el-input size="small" v-model="searchParam.title"></el-input>-->
<!--            </el-form-item>-->
            <el-form-item label="路径/文件夹">
                <el-input size="small" v-model="searchParam.filePath"></el-input>
            </el-form-item>
            <el-form-item label="分类">
                <el-select
                    v-model="searchParam.category"
                    filterable
                    clearable
                    allow-create
                    placeholder="请选择分类">
                    <el-option
                        v-for="item in categoryOptions"
                        :key="item"
                        :label="item"
                        :value="item">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="状态">
                <el-select v-model="searchParam.status" clearable placeholder="请选择">
                    <el-option
                        v-for="item in statusOptions"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="init" size="small" icon="el-icon-search">查询</el-button>
            </el-form-item>
        </el-form>

<!--        <el-button type="primary" size="small" icon="el-icon-plus" @click="openAddLinkItemDialog">新增直链</el-button>-->
<!--        <el-button type="danger" size="small" icon="el-icon-delete" @click="batchDeleteLinkItem">删除所选</el-button>-->
        <el-button type="primary" size="small" icon="el-icon-time" @click="timedTask(1)">开启定时任务</el-button>
        <el-button type="danger" size="small" icon="el-icon-switch-button" @click="timedTask(0)">关闭定时任务</el-button>
        <el-table
            :data="linkLog"
            @row-click="handleRowClick"
            @sort-change="sortMethod"
            :default-sort = "{prop: 'createTime', order: 'descending'}"
            ref="shortLinkTable">
            <el-table-column
                type='selection'
                label="序号">
            </el-table-column>
            <el-table-column
                prop="id"
                width="50"
                label="序号">
            </el-table-column>
            <el-table-column
                prop="title"
                :show-overflow-tooltip="true"
                sortable="custom"
                :sort-orders="['ascending', 'descending']"
                label="标题">
<!--                <template slot-scope="scope">-->
<!--                    <a target="_blank" :href="siteDomain + '/s/' + scope.row.key">{{scope.row.key}}</a>-->
<!--                    <i class="el-icon-edit-outline table-edit-icon" @click="editKey(scope.row.id)"></i>-->
<!--                </template>-->
            </el-table-column>
            <el-table-column
                prop="driveId"
                width="100"
                :show-overflow-tooltip="true"
                sortable="custom"
                :sort-orders="['ascending', 'descending']"
                label="驱动Id">
            </el-table-column>
            <el-table-column
                prop="category"
                :show-overflow-tooltip="true"
                width="100"
                sortable="custom"
                :sort-orders="['ascending', 'descending']"
                label="分类">
            </el-table-column>
            <el-table-column
                prop="filePath"
                :show-overflow-tooltip="true"
                sortable="custom"
                :sort-orders="['ascending', 'descending']"
                label="路径">
            </el-table-column>
            <el-table-column
                prop="createTime"
                width="150"
                sortable="custom"
                :sort-orders="['ascending', 'descending']"
                label="创建日期">
            </el-table-column>
            <el-table-column
                prop="status"
                width="100"
                sortable="custom"
                :sort-orders="['ascending', 'descending']"
                label="状态">
                <template scope="scope">
                    <el-link  type="success" size="mini" @click="copyDirStr(scope.row)" v-if="scope.row.status==1">已完成</el-link>
                    <el-link  type="info" size="mini" @click="ergodicDir(scope.row)" v-if="scope.row.status==0">未遍历</el-link>
                    <el-link  type="warning" size="mini" @click="ergodicDir(scope.row)" v-if="scope.row.status==2">异常</el-link>
                </template>
            </el-table-column>
            <el-table-column
                width="120"
                label="操作">
                <template slot-scope="scope">
                    <el-popconfirm
                        @confirm="deleteLink(scope.row.id)"
                        confirmButtonText='确定'
                        cancelButtonText='取消'
                        icon="el-icon-info"
                        iconColor="red"
                        title="确定删除吗?">
                        <el-button slot="reference" class="el-icon-delete" size="mini" type="danger">删除</el-button>
                    </el-popconfirm>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination background
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :current-page="searchParam.page"
                       :page-sizes="[10, 50, 100, 200, 999999]"
                       layout="total, sizes, prev, pager, next, jumper"
                       :page-size="searchParam.limit"
                       :total="searchParam.total">
        </el-pagination>

        <el-dialog id="cacheDialog" width="50%" title="直链管理" :modal-append-to-body="false" v-if="addLinkVisible" :visible.sync="addLinkVisible">
            <el-form :model="addLinkModel" label-width="150px">
                <el-form-item label="驱动器 ID">
                    <el-select v-model="addLinkModel.driveId" placeholder="请选择驱动器">
                        <el-option v-for="item in driveList"
                                   :key="item.id"
                                   :label="item.name"
                                   :value="item.id">
                            <span style="float: left">{{ item.name }}</span>
                            <span style="float: right; color: #8492a6; font-size: 13px">{{item.id}}</span>
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="路径">
                    <el-input v-model="addLinkModel.path"  placeholder="请输入路径地址" class="input-with-select">
                        <div slot="prepend">/directlink/{{addLinkModel.driveId}}</div>
                    </el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" icon="el-icon-check" size="small" @click="addLinkItemAction">保存</el-button>
                <el-button icon="el-icon-close" size="small" @click="addLinkVisible = false">关闭</el-button>
            </div>
        </el-dialog>
    </el-card>
</template>

<script>
export default {
    name: "ShortLink",
    data() {
        return {
            addLinkModel: {
                driveId: null,
                path: ''
            },
            addLinkVisible: false,
            linkUrl: '',
            linkLog: [],
            searchParam: {
                page: 1,
                limit: 10,
                total: 0,
                filePath: '',
                category: '',
                title: '',
                status: '',
                orderBy: '',
                orderDirection: '',
            },
            statusOptions:[{value: 0,label:'未遍历'},{value: 1,label:'已完成'}],
            categoryOptions:[],
            pickerOptions: {
                shortcuts: [{
                    text: '最近一周',
                    onClick(picker) {
                        const end = new Date();
                        const start = new Date();
                        start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                        picker.$emit('pick', [start, end]);
                    }
                }, {
                    text: '最近一个月',
                    onClick(picker) {
                        const end = new Date();
                        const start = new Date();
                        start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                        picker.$emit('pick', [start, end]);
                    }
                }, {
                    text: '最近三个月',
                    onClick(picker) {
                        const end = new Date();
                        const start = new Date();
                        start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
                        picker.$emit('pick', [start, end]);
                    }
                }]
            },
            date: ['', ''],
            driveList: [],
            siteDomain: ''
        }
    },
    methods: {
        editKey(id) {
            this.$prompt('请输入要修改为的 Key。', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消'
            }).then(({value}) => {
                this.$http.get('admin/api/short-link/key', {params: {id: id, newKey: value}}).then(() => {
                    this.$message({
                        message: '修改成功',
                        type: 'success'
                    });
                    this.init();
                });
            });
        },
        findCategory(){
            this.$http.get('api/ergodic/category', ).then((response) => {
                if (response.data.code === 0) {
                    this.categoryOptions =response.data.data;
                }
            });
        },
        copyDirStr(row){
            if (!row.typechoVideoStr || !row.status) {
                this.$message.error('复制失败');
                return;
            }
            console.info(row.typechoVideoStr);
            this.$copyText(row.typechoVideoStr).then(() => {
                this.$message.success('复制成功');
            }, () => {
                this.$message.error('复制失败');
            });
        },
        ergodicDir(row){
            // this.$message.success(row.typechoVideoStr);
            this.$http.post('admin/ergodic/' + row.id).then(() => {
            });
            this.$message.success('开始获取目录下文件Url');
        },
        timedTask(flag){
          if (flag) flag ='start'
            this.$http.get('admin/ergodic/job',{params: {status:flag}}).then((response) => {
                this.$message.success(response.data.msg);
            });
        },
        //点击行触发，选中或不选中复选框
        handleRowClick(row){
            this.$refs.shortLinkTable.toggleRowSelection(row);
        },
        batchDeleteLinkItem() {
            let selection = this.$refs.shortLinkTable.selection;

            if (selection.length === 0) {
                this.$message.warning('请至少选中一行数据');
                return;
            }

            this.$confirm('删除后不可恢复, 是否确认删除', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                callback: action => {
                    if (action === 'confirm') {

                        let arr = [];
                        for (let index in selection) {
                            arr[index] = selection[index].id;
                        }

                        this.$http.delete('admin/api/short-link', {params: {id: arr}}).then((response) => {
                            if (response.data.code === 0) {
                                this.$message.success('删除成功');
                                this.init();
                            } else {
                                this.$message.error(response.data.msg);
                            }
                        });
                    }
                }
            });
        },
        addLinkItemAction() {
            alert(this.$store.getters.domain);
            this.addLinkModel.path = this.common.removeDuplicateSeparator('/' + this.addLinkModel.path);
            this.$http.get('api/short-link', {params: this.addLinkModel}).then(() => {
                this.$message.success('添加成功');
                this.init();
                this.addLinkVisible = false;
            });
        },
        openAddLinkItemDialog() {
            this.addLinkVisible = true;
        },
        dateChange() {
            if (this.date) {
                this.searchParam.dateFrom = this.date[0];
                this.searchParam.dateTo = this.date[1];
            } else {
                this.searchParam.dateFrom = "";
                this.searchParam.dateTo = "";
            }
        },
        sortMethod({prop, order}) {
            this.searchParam.orderBy = prop;
            this.searchParam.orderDirection = order === "descending" ? "desc" : "asc";
            this.init();
        },
        handleSizeChange(val) {
            this.searchParam.limit = val;
            this.searchParam.page = 1;
            this.init();
        },
        handleCurrentChange(val) {
            this.searchParam.page = val;
            this.init();
        },
        deleteLink(id) {
            this.$http.delete(`admin/ergodic/${id}`).then((response) => {
                if (response.data.code === this.common.responseCode.SUCCESS) {
                    this.$message.success('删除成功');
                    this.init();
                } else {
                    this.$message.error(response.data.msg);
                }
            });
        },
        init() {
            this.$http.get('admin/ergodics', {params: this.searchParam}).then((response) => {
                this.linkLog = response.data.data.content;
                this.searchParam.total = response.data.data.totalElements;
                this.searchParam.limit = response.data.data.size;
            });
        },
        loadDriveList() {
            this.$http.get('api/drive/list').then((response) => {
                this.driveList = response.data.data;
            });
        },
        getDomainId() {
            this.$http.get('admin/config').then((response) => {
                this.siteDomain = response.data.data.domain;
            });
        },
    },
    mounted() {
        this.findCategory();
        this.init();
        this.loadDriveList();
        this.getDomainId();
    }
}
</script>

<style scoped>
    .zfile-admin-short-form >>> .el-form-item:first-child {
        margin-left: 10px;
    }

    .zfile-admin-short-form >>> .el-form-item:not(:first-child) {
        margin-left: 20px;
    }

    .el-pagination {
        margin-top: 15px;
    }

    .table-edit-icon {
        margin-left: 10px;
        cursor: pointer;
    }
</style>
