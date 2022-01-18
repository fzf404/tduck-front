<template>
    <div class="my-project-container">
        <div class="result-table-view">
            <el-button type="success" style="margin-bottom: 10px" plain @click="getJoinCode">
                生成加入码
            </el-button>
            <el-table
                :data="subUserData"
                header-cell-class-name="data-table-header"
                stripe
            >
                <el-table-column
                    header-align="center"
                    align="center"
                    prop="userId"
                    label="用户ID"
                />

                <el-table-column
                    header-align="center"
                    align="center"
                    prop="nickName"
                    label="姓名"
                />

                <el-table-column
                    header-align="center"
                    align="center"
                    label="操作"
                >
                    <template slot-scope="scope">
                        <el-button size="small" type="text" @click="deleteSubUser(scope.row.subUser)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
    </div>
</template>
<script>
import {Message} from 'element-ui'

export default {
    name: 'SubUser',
    data() {
        return {
            subUserData: null
        }
    },
    created() {
        this.getAllSubUser()
    },
    methods: {
        // 请求全部子用户
        getAllSubUser() {
            this.$api.get('/user/relation/generateJoinCode').then(res => {
                this.joinCode = res.data.joinCode
            })
            this.$api.get('/user/relation/show_sub_user_list').then(res => {
                this.subUserData = res.data
            })
        },
        deleteSubUser(subUser) {
            this.$api.post('/user/relation/del', {subUser: subUser}).then(
                res => {   
                    Message({
                        message: res.msg,
                        type: 'success',
                        duration: 3 * 1000
                    })
                    this.getAllSubUser()
                }
            )
        },
        getJoinCode() {
            this.$alert(this.joinCode, '加入码', {
                confirmButtonText: '复制到剪贴板'
            }).then(() => {
                this.$copyText(this.joinCode).then(
                    () => {
                        Message({
                            message: '复制成功',
                            type: 'success',
                            duration: 3 * 1000
                        })
                    },
                    () => {
                        Message({
                            message: '复制失败，请手动复制',
                            type: 'danger',
                            duration: 3 * 1000
                        })
                    },
                )
            })
        }
    }
}
</script>

<style lang="scss" scoped>
.my-project-container {
    margin-top: 20px;
    width: 100%;
    height: 100%;  
}

.result-table-view {
  width: 80%;
  min-height: 75vh;
  margin: 6px auto;
}
</style>
