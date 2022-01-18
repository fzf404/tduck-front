<template>
    <div class="my-project-container">
        <!-- 布局切换 -->
        <div class="filter-view fr">
            <div>
                <span @click="switchDataShowTypeHandle('gird')">
                    <font-icon
                        :class="{'show-view-type-icon-active':dataShowType=='gird'}"
                        class="fas fa-th-large show-view-type-icon "
                    />
                </span>
                <span @click="switchDataShowTypeHandle('table')">
                    <font-icon
                        :class="{'show-view-type-icon-active':dataShowType=='table'}"
                        class="fas  fa-th-list show-view-type-icon"
                    />
                </span>
            </div>
        </div>
        <!-- gird 布局展示 -->
        <div v-if="dataShowType=='gird'" class="project-grid-container">
            <div
                v-if="projectList.length"
                v-loading="projectListLoading"
                class="project-grid-view"
            >
                <div v-for="p in projectList" :key="p.id" class="project-grid-item-view pointer">
                    <el-row align="middle" justify="center" type="flex">
                        <el-col :span="5">
                            <span
                                :style="getStatusColorStyle(p.status)"
                                class="project-grid-view-status"
                            />
                        </el-col>
                        <el-col :span="19">
                            <el-tooltip :content="p.name" placement="top">
                                <p class="project-title">
                                    {{ p.name }}
                                </p>
                            </el-tooltip>
                        </el-col>
                    </el-row>
                    <img class="project-grid-view-preimg"
                         src="data:image/png;base64,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"
                    >
                    <p class="project-grid-view-time">创建时间：{{ p.createTime | formatDate }}</p>
                    <div class="gird-operating-btns">
                        <el-button type="text" @click="writeHandle(p.key)">
                            <i class="el-icon-edit" />
                            填写
                        </el-button>
                        <span>
                            <el-button
                                v-if="p.status!=1" type="text" @click="getRate(p.key)"
                            >
                                <i class="el-icon-data-analysis" />
                                查看评分
                            </el-button>
                        </span>
                    </div>
                </div>
            </div>
            <div v-if="!projectListLoading&&projectList.length==0">
                <data-empty />
            </div>
        </div>
        <!-- table 格式 -->
        <div v-if="dataShowType=='table'" class="project-table-view">
            <el-table
                :data="projectList"
                border
                empty-text="暂无数据"
                highlight-current-row

                stripe
                style="width: 100%;"
            >
                <el-table-column
                    align="center"
                    label="标题"
                    prop="name"
                    show-overflow-tooltip
                />
                <el-table-column
                    align="center"
                    label="描述"
                    prop="describe"
                    show-overflow-tooltip
                />
                <el-table-column
                    align="center"
                    label="状态"
                >
                    <template slot-scope="scope">
                        <span v-for="status in projectStatusList" :key="status.code">
                            <span
                                v-if="status.code==scope.row.status"
                            >
                                {{ status.name }}
                            </span>
                        </span>
                    </template>
                </el-table-column>
                <el-table-column
                    align="center"
                    label="创建时间"
                    prop="createTime"
                />
                <el-table-column
                    align="center"
                    label="更新时间"
                    prop="updateTime"
                />
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button type="text"
                                   @click="writeHandle(scope.row.key)"
                        >
                            填写
                        </el-button>
                        &ensp;
                        <span>
                            <el-button
                                v-if="scope.row.status!=1"
                                class="green-text-btn"
                                type="text"
                                @click="getRate(scope.row.key)"
                            >
                                查看评分
                            </el-button>
                        </span>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <!-- 填写二维码 -->
        <el-dialog title="扫码填写" :visible.sync="qrCodeVisible" width="240px">
            <vue-qr v-if="writeLink" :size="194" :text="writeLink" />
        </el-dialog>
        <!-- 评分内容查看 -->
        <el-dialog title="评分详情" :visible.sync="rateVisible" width="400px">
            <el-rate
                v-model="rateData.score"
                disabled
                show-score
                text-color="#ff9900"
            />
            <el-divider />
            <el-input
                v-model="rateData.comment" 
                type="textarea"
                :rows="3"
                disabled
            />
        </el-dialog>
    </div>
</template>
<script>
import dayjs from 'dayjs'
import VueQr from 'vue-qr'
import {Message} from 'element-ui'

let projectStatusList = [
    {code: 1, name: '未发布', color: '#006EFF'},
    {code: 2, name: '收集中', color: '#34C82A'},
    {code: 3, name: '已结束', color: '#955A45'}
]

export default {
    name: 'TODO',
    components: {
        VueQr
    },
    filters: {
        formatDate(time) {
            return dayjs(time).format('YYYY/MM/DD')
        }
    },
    data() {
        return {
            dataShowType: 'table',
            // total: 0,
            // queryParams: {
            //     current: 1,
            //     size: 10,
            //     name: '',
            //     beginDateTime: null,
            //     endDateTime: null,
            //     status: null
            // },

            // 填写地址
            writeLink: null,
            // 模态框展示
            qrCodeVisible: false,
            // 评分查看
            rateVisible: false,
            rateData: {
                score: 0,
                comments: ''
            },
            projectStatusList: projectStatusList,
            projectList: [],
            projectListLoading: true
        }
    },
    computed: {
        getStatusColorStyle() {
            return function(code) {
                let color = this.projectStatusList.find(item => item.code == code).color
                return {
                    backgroundColor: color,
                    borderColor: color
                }
            }
        }
    },
    created() {
        this.getAllSubUser()
    },
    methods: {
        // 切换展示方式
        switchDataShowTypeHandle(type) {
            this.dataShowType = type
        },

        // 请求全部问卷
        getAllSubUser() {
            this.$api.get('/user/score/get_parent_user_projects').then(res => {
                this.projectList = res.data
                this.projectListLoading = false
            })
        },

        // 填写问卷二维码
        writeHandle(projectKey) {
            const url = `${window.location.protocol}//${window.location.host}/s/${projectKey}`
            this.writeLink = url
            this.qrCodeVisible = true
        },
        // 获取评分内容
        getRate(projectKey) {
            Message({
                message: '请求中',
                type: 'warning',
                duration: 2 * 1000
            })
            this.$api.get('/user/score/get_score', {params: {'project_key': projectKey}}).then(
                res => { 
                    this.rateData = res.data 
                    this.rateVisible = true
                })
        }
    }
}
</script>

<style lang="scss" scoped>
.my-project-container {
    display: flex;
    width: 100%;
    height: 100%;
    flex-direction: column;
    align-items: center;
    align-content: center;
}
.back-view {
    display: flex;
    width: 80%;
    align-content: flex-start;
    margin: 10px;
}
.filter-view {
    margin-top: 20px;
    display: flex;
    flex-direction: row;
}
.show-view-type-icon {
    color: white;
    font-size: 18px;
    -webkit-text-stroke: 0.5px #a8a8a8;
    margin: 5px;
    cursor: pointer;
}
.show-view-type-icon-active {
    color: rgba(92, 155, 249, 100);
    -webkit-text-stroke: 0.5px rgba(92, 155, 249, 100);
}
.project-grid-container {
    margin-top: 20px;
    display: flex;
    width: 100%;
    justify-content: center;
}
.project-grid-view {
    display: flex;
    width: 950px;
    flex-direction: row;
    flex-wrap: wrap;
}
.project-table-view {
    margin-top: 20px;
    width: 80%;
}
.project-grid-item-view {
    width: 169px;
    height: 199px;
    line-height: 20px;
    border-radius: 4px;
    background-color: rgba(255, 255, 255, 100);
    text-align: center;
    margin: 10px;
    position: relative;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
    .project-title {
        color: rgba(16, 16, 16, 100);
        font-size: 14px;
        text-align: left;
        line-height: 20px;
        max-height: 20px;
        -webkit-box-orient: vertical;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    .project-grid-view-status {
        display: inline-block;
        width: 7px;
        height: 7px;
        line-height: 20px;
        background-color: rgba(0, 110, 255, 100);
        text-align: center;
        border: 1px solid rgba(0, 110, 255, 100);
        border-radius: 20px;
    }
}
.gird-operating-btns {
    position: absolute;
    width: 100%;
    margin: 0;
    padding: 0;
    background-color: #f0f0f0;
    bottom: 0;
    display: none;
    border: none;
}
.project-grid-item-view:hover .gird-operating-btns {
    display: block;
}
.project-grid-view-preimg {
    width: 143px;
    height: 121px;
}
.project-grid-view-time {
    color: rgba(144, 147, 153, 100);
    font-size: 12px;
    line-height: 20px;
    text-align: center;
    margin: 0;
}
.project-page-view {
    margin-top: 30px;
}
</style>
