<template>
    <div class="statistics-tabs">
        <el-tabs v-model="activeName" type="border-card">
            <el-tab-pane label="列表视图" name="list">
                <list />
            </el-tab-pane>
        </el-tabs>
    </div>
</template>

<script>
import list from './list'

export default {
    name: 'ProjectCheck',
    components: {
        list
    },
    data() {
        return {
            activeName: 'list'
        }
    },
    mounted() {

    }, methods: {}

}
</script>

<style scoped>
.statistics-tabs {
  padding: 20px;
  height: 100%;
}

::v-deep .el-tabs__content {
  padding: 0;
}
</style>
