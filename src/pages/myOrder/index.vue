<template>
    <div class="common_width dm_myOrder">
        <TableTitle title='我的订单'>
            <div>（当前共有 <i>{{ tableData.length }}</i> 笔订单）</div>
        </TableTitle>
        <el-table
            border
            style="width: 100%"
            size='small'
            :class="`dm_myOrder__title${tableData.length ? ' hidden' : ''}`"
        >
            <el-table-column v-for='c in columns' :key='c.id'
                :prop="c.prop"
                :label="c.label"
                :width="c.width"
                align='center'
            ></el-table-column>
            
            <el-table-column
                prop="operation"
                label="操作"
                width="140"
                align='center'
            >
            </el-table-column>
        </el-table>

        <div v-for='(tb, index) in tableData' :key='tb.id' class="dm_myOrder__content">
            <el-row class='t_header'>
                <el-col :span='6'>{{ tb.submitTime }}</el-col>
                <el-col :span='6'>订单号： {{ tb.ordernum }}</el-col>
                <el-col :span='12'>
                    <span @click="deleteCartClick(tb.id)">删</span>
                </el-col>
            </el-row>
            <el-table
                :data="tb.content"
                border
                style="width: 100%"
                size='small'
                :show-header='false'
            >
                <el-table-column v-for='c in columns' :key='c.id'
                    :prop="c.prop"
                    :label="c.label"
                    :width="c.width"
                    align='center'
                >
                    <template slot-scope="scope">
                        <img v-if="c.prop == 'mainPicture'" style='width: 100%' :src="$url + scope.row.mainPicture" />
                        <router-link v-if="c.prop == 'description'" class='title_style' :to="{ name: 'productsDetails', params: { id: scope.row.id } }">
                            <span :title='scope.row.description'>{{ scope.row.description }}</span>
                            <span class='ellipsis'>规格：{{ scope.row.spec }}</span>
                        </router-link>
                        <span v-if="c.prop == 'price' || c.prop == 'totalprice'">￥{{ c.prop == 'price' ? scope.row['price'] : c.prop == 'totalprice' ? scope.row['totalprice'] : 0 | keepTwoDecimalPlaces }}</span>
                        <span v-if="c.prop == 'num'">x{{ scope.row.num }}</span>
                    </template>
                </el-table-column>
                
                <el-table-column
                    prop="operation"
                    label="操作"
                    width="140"
                    align='center'
                >
                    <template>             
                        <div class='operation'>
                            <span @click="goNextPage('comment', tb.id, index)">评价</span>
                            <span @click="goNextPage('detail', tb.id)">订单详情</span>
                        </div>
                    </template>
                </el-table-column>
            </el-table>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            columns: [
                { id: 1, prop: 'mainPicture', label: '主图', width: 107 },
                { id: 2, prop: 'description', label: '商品详情' },
                { id: 3, prop: 'price', label: '单价', width: 162 },
                { id: 4, prop: 'num', label: '数量', width: 130 },
                { id: 5, prop: 'totalprice', label: '小计', width: 172 }
            ],
            tableData: []
        }
    },
    mounted() {
        this.getAllOrderData();
    },
    methods: {
        // 进入 评价/订单详情
        goNextPage(_this, id, index) {
            let obj = {
                comment: () => {
                    this.$router.push({ name: 'myComment', query: { id, oIndex: index } });
                },
                detail: () => {
                    this.$router.push({ name: 'orderDetails', query: { id } });
                }
            };
            obj[_this]();
        },
        // 删除 - 操作
        deleteCartClick(id) {
            this.$confirm('确认删除这笔订单?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.postDeleteOrderData({ id });
            }).catch(() => {
                return;
            })
        },
        // 查询我的订单
        async getAllOrderData() {
            const res = await this.$service.getAllOrderData({
                uname: sessionStorage.getItem('uname')
            });
            try {
                if (res.data.code === 200) {
                    const { products=[] } = res.data.data || {};
                    this.tableData = products;
                }
            } catch (err) {
                console.log(err);
            }
        },
        // 删除
        async postDeleteOrderData(params = {}) {
            const res = await this.$service.postDeleteOrderData(params);
            try {
                if (res.data.code === 200) {
                    this.getAllOrderData();
                }
            } catch (err) {
                console.log(err);
            }
        }
    }
}
</script>

<style lang="less">
    @import './index.less';
</style>