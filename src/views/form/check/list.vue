<template>
    <div class="statistics-container">
        <div class="result-table-view">
            <!-- 表头绑定 -->
            <el-table
                :data="subUserStatus"
                header-cell-class-name="data-table-header"
                stripe
            >
                <el-table-column
                    prop="nickname"
                    label="姓名"
                />

                <el-table-column
                    prop="status"
                    label="填写状态"
                >
                    <template slot-scope="scope">
                        <el-tag
                            :type="scope.row.status ? 'primary': 'danger'"
                            disable-transitions
                        > 
                            {{ scope.row.status ? '已提交' : '未提交' }} 
                        </el-tag>
                    </template>
                </el-table-column>

                <el-table-column
                    prop="score_status"
                    label="评分状态"
                >
                    <template slot-scope="scope">
                        <el-tag
                            :type="scope.row.score_status ? 'success' : 'warning'"
                            disable-transitions
                        > 
                            {{ scope.row.score_status ? '已评分' : '未评分' }} 
                        </el-tag>
                    </template>
                </el-table-column>

                <el-table-column
                    :render-header="renderHeader"
                    fixed="right"
                    width="300"
                >
                    <template slot-scope="scope">
                        <!-- <el-button size="small" type="text" @click="openDetailDrawerHandle(scope.row)">查看</el-button> -->
                        <el-button :disabled="!scope.row.status" size="small" type="text" @click="openCheckDrawerHandle(scope.row)">评分</el-button>
                    </template>
                </el-table-column>
            </el-table>
            
            <!-- 提交信息查看 -->
            <el-drawer
                v-if="activeResultRow"
                :visible.sync="checkDrawer"
                :with-header="false"
            >
                <el-scrollbar style="height: 100%;">
                    <el-card class="detail-container">
                        <!-- 提交详情 -->
                        <div class="clearfix" style="font-size: 20px;">
                            <span>提交详情</span>
                        </div>
                        <el-divider />
                        
                        <div>
                            <div v-for="item in projectItemList" :key="item.id">
                                <h4>{{ item.label }}</h4>
                                <result-item :field-item-id="item.formItemId" :project-item-data="item"
                                             :result-data="activeResultRow"
                                />
                                <el-divider />
                            </div>
                        </div>
                        <div class="clearfix" style="font-size: 20px;">
                            <span>评分</span>
                        </div>
                        <el-divider />
                        <div>
                            <el-rate v-model="checkData.score" />
                        </div>
                        <el-divider />
                        <el-input
                            v-model="checkData.comment" 
                            type="textarea"
                            :rows="3"
                            placeholder="请输入内容" 
                        />
                        <el-divider />
                        <el-button type="primary" plain @click="sendCheckData">提交</el-button>
                    </el-card>
                </el-scrollbar>
            </el-drawer>
        </div>
    </div>
</template>

<script>
import _ from 'lodash'
import ResultItem from './item'
import {Message} from 'element-ui'

export default {
    name: 'ProjectStatisticsList',
    components: {
        ResultItem
        // DataFilter
    },
    data() {
        return {
            projectKey: null,
            customFilterDialogVisible: false,
            customColumnDialogVisible: false,
            checkedFilterColumns: [],
            // // 固定自定义列 如序号等
            // fixedCustomColumns: fixedDefaultFormColumn,
            // fixedFormTailColumns: fixedDefaultFormTailColumn,
            // // 选中的
            // checkedFixedCustomColumns: fixedDefaultFormColumn,
            // fixedDefaultLabelFormColumn: fixedDefaultLabelFormColumn,
            // checkedFixedTailCustomColumns: fixedDefaultFormTailColumn,
            // fixedDefaultLabelFormTailColumn: fixedDefaultLabelFormTailColumn,
            // 自定义表单属性
            // checkOtherCustomColumns: [],
            // otherCustomColumns: [],
            projectResultList: [],
            projectItemList: [],
            projectData: null,
            projectItemColumns: {},
            // 填写信息侧边栏
            // detailDrawer: false,
            // 填写信息详情
            activeResultRow: null,
            // 评分侧边栏
            checkDrawer: false,
            // 评分详情
            checkData: {
                projectKey: null,
                subUser: null,
                score: null,
                comment: null
            },
            // 下级用户状态
            subUserStatus: []
        
        }
    },
    mounted() {
        this.projectKey = this.$route.query.key
        this.queryProjectResult()
        this.queryProjectItems()
        this.queryProject()
    }, 
    methods: {
        renderHeader() {
            return (
                <i class="el-icon-setting" style="color:currentColor"
                    onClick={() => this.customColumnDialogVisible = true}></i>
            )
        },
        queryProject() {
            this.$api.get(`/user/project/${this.projectKey}`).then(res => {
                this.projectData = res.data
            })
        },
        queryProjectResult() {
            // 请求全部信息
            this.$api.get('/user/score/show_sub_user_status', {params: {'project_key': this.projectKey}}).then(res => {
                let { subUserStatus } = res.data
                this.subUserStatus = subUserStatus
            })
        },
        // 模板
        queryProjectItems() {
            this.$api.get('/user/project/item/list', {params: {key: this.projectKey, displayType: false}}).then(res => {
                if (res.data) {
                    res.data.map(item => {
                        _.set(this.projectItemColumns, `field${item.formItemId}`, item.label)
                    })
                }
                this.projectItemList = res.data
                // this.getDbCheckedColumns()
            })
        },
        // 打开评分框
        openCheckDrawerHandle(row) {
            this.checkData.projectKey = this.projectKey
            this.checkData.subUser = row.record.wxOpenId
            this.activeResultRow = row.record
            this.checkDrawer = true
        },
        // 发送评分数据
        sendCheckData() { 
            this.$api.post('/user/score', this.checkData).then(res => {
                Message({
                    message: res.msg,
                    type: 'success',
                    duration: 5 * 1000
                })
            })
            this.queryProjectResult()
        }
    }

}
</script>

<style scoped>
.statistics-container {
  margin-top: 20px;
  width: 100%;
  height: 100%;
}

.custom-col-container >>> .el-checkbox__label {
  width: 200px;
  min-height: 25px;
  line-height: 25px;
  vertical-align: middle;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.result-table-view {
  width: 80%;
  min-height: 75vh;
  margin: 6px auto;
}

.detail-container {
  padding: 20px;
  height: 100% !important;
}

.filter-table-view {
  width: 80%;
  margin: 0 auto;
}

::v-deep .el-icon-setting {
  font-size: 24px;
  line-height: 25px;
  color: white;
}

::v-deep .data-table-header .cell {
  text-overflow: ellipsis !important;
  white-space: nowrap !important;
}

/* 1.显示滚动条：当内容超出容器的时候，可以拖动： */
::v-deep .el-drawer__body {
  overflow: auto;

  /* overflow-x: auto; */
}

/* 2.隐藏滚动条，太丑了 */
::v-deep .el-drawer__container ::-webkit-scrollbar {
  display: none;
}

</style>
