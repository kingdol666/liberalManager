<template>
    <div class="home" style="padding: 10px">
        <!-- 只有管理员可以访问 -->
        <div v-if="user.role == 1">
            <!-- 搜索-->
            <div style="margin: 10px 0;">

                <el-form inline="true" size="small">
                    <el-form-item label="图书编号">
                        <el-input v-model="search1" placeholder="请输入图书编号" clearable>
                            <template #prefix><el-icon class="el-input__icon">
                                    <search />
                                </el-icon></template>
                        </el-input>
                    </el-form-item>
                    <el-form-item label="图书名称">
                        <el-input v-model="search2" placeholder="请输入图书名称" clearable>
                            <template #prefix><el-icon class="el-input__icon">
                                    <search />
                                </el-icon></template>
                        </el-input>
                    </el-form-item>
                    <el-form-item label="借阅者">
                        <el-input v-model="search3" placeholder="请输入借阅者昵称" clearable>
                            <template #prefix><el-icon class="el-input__icon">
                                    <search />
                                </el-icon></template>
                        </el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" style="margin-left: 1%" @click="load" size="mini">查询</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button size="mini" type="danger" @click="clear">重置</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <!-- 按钮-->
            <div style="margin: 10px 0;">
                <el-popconfirm title="确认归还?" @confirm="deleteBatch">
                    <template #reference>
                        <el-button type="danger" size="mini">批量归还</el-button>
                    </template>
                </el-popconfirm>
            </div>
            <!-- 数据字段-->
            <el-table :data="tableData" stripe border="true" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55">
                </el-table-column>
                <el-table-column prop="isbn" label="图书编号" sortable />
                <el-table-column prop="bookName" label="图书名称" />
                <el-table-column prop="nickName" label="借阅者" />
                <el-table-column prop="lendtime" label="借阅时间" />
                <el-table-column prop="deadtime" label="最迟归还日期" />
                <el-table-column prop="prolong" label="可续借次数" />
                <el-table-column fixed="right" label="操作">
                    <template v-slot="scope">
                        <el-button size="mini" @click="handleEdit(scope.row)">修改</el-button>
                        <el-popconfirm title="确认归还?" @confirm="handleDelete(scope.row)">
                            <template #reference>
                                <el-button type="danger" size="mini">归还</el-button>
                            </template>
                        </el-popconfirm>
                    </template>
                </el-table-column>
            </el-table>
            <!--    分页-->
            <div style="margin: 10px 0">
                <el-pagination v-model:currentPage="currentPage" :page-sizes="[5, 10, 20]" :page-size="pageSize"
                    layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
                    @current-change="handleCurrentChange">
                </el-pagination>

                <el-dialog v-model="dialogVisible2" title="修改借阅信息" width="30%">
                    <el-form :model="form" label-width="120px">

                        <el-form-item label="图书编号">
                            <el-input style="width: 80%" v-model="form.isbn"></el-input>
                        </el-form-item>
                        <el-form-item label="图书名称">
                            <el-input style="width: 80%" v-model="form.bookName"></el-input>
                        </el-form-item>
                        <el-form-item label="借阅者">
                            <el-input style="width: 80%" v-model="form.nickName"></el-input>
                        </el-form-item>
                        <el-form-item label="续借次数">
                            <el-input style="width: 80%" v-model="form.prolong"></el-input>
                        </el-form-item>
                    </el-form>
                    <template #footer>
                        <span class="dialog-footer">
                            <el-button @click="dialogVisible2 = false">取 消</el-button>
                            <el-button type="primary" @click="save">确 定</el-button>
                        </span>
                    </template>
                </el-dialog>

            </div>
        </div>
        <!-- 非管理员提示 -->
        <div v-else class="no-permission">
            <el-alert title="权限不足" message="您没有访问该页面的权限，请联系管理员" type="warning" :closable="false" show-icon />
        </div>
    </div>
</template>

<script>
// @ is an alias to /src
import request from "../utils/request";
import { ElMessage } from "element-plus";
import moment from "moment";
import router from "@/router";
export default {
    created() {
        let userStr = sessionStorage.getItem("user") || "{}"
        this.user = JSON.parse(userStr)
        let userJson = sessionStorage.getItem("user")
        if (!userJson) {
            router.push("/login")
        }
        this.load()
    },
    name: 'bookwithuser',
    methods: {

        handleSelectionChange(val) {
            this.forms = val
        },
        deleteBatch() {
            if (!this.forms.length) {
                ElMessage.warning("请选择数据！")
                return
            }
            //  一个小优化，直接发送这个数组，而不是一个一个的提交归还
            request.post("bookwithuser/deleteRecords", this.forms).then(res => {
                if (res.code === '0') {
                    ElMessage.success("批量归还成功")
                    // 批量更新借阅记录状态为已归还和图书状态
                    const updatePromises = []

                    this.forms.forEach(row => {
                        // 借阅记录更新请求
                        const returnBookRequest = {
                            status: '1', // 设置为已归还状态
                            returnTime: new Date().toISOString().slice(0, 19).replace('T', ' '), // 设置当前时间为归还时间
                            // 只包含RequestLendRecord.java中定义的字段
                        }
                        // 调用借阅记录更新接口
                        updatePromises.push(request.put(`LendRecord/${row.isbn}`, returnBookRequest))

                        // 图书状态更新请求
                        // 直接返回请求，不使用嵌套的Promise.all
                        updatePromises.push(request.put(`book/status?isbn=${row.isbn}&status=1`))
                    })

                    // 等待所有更新请求完成
                    Promise.all(updatePromises).then(results => {
                        console.log('所有更新请求完成：', results)
                        // 检查是否所有更新都成功
                        const allUpdateSuccess = results.every(item => item.code === 0)
                        if (allUpdateSuccess) {
                            ElMessage.success("所有记录更新成功")
                        } else {
                            ElMessage.warning("部分记录更新可能失败，请检查")
                        }
                        this.load()
                    }).catch(error => {
                        console.error('更新请求失败：', error)
                        ElMessage.error("更新请求失败，请稍后重试")
                        this.load()
                    })
                }
                else {
                    ElMessage.error(res.msg)
                    this.load()
                }
            })
        },
        load() {
            if (this.user.role == 1) {
                request.get("/bookwithuser", {
                    params: {
                        pageNum: this.currentPage,
                        pageSize: this.pageSize,
                        search1: this.search1,
                        search2: this.search2,
                        search3: this.search3,
                    }
                }).then(res => {
                    console.log(res)
                    this.tableData = res.data.records
                    this.total = res.data.total
                })
            }
        },
        clear() {
            this.search1 = ""
            this.search2 = ""
            this.search3 = ""
            this.load()
        },
        handleDelete(row) {
            const form3 = JSON.parse(JSON.stringify(row))
            // 1. 执行书籍归还操作
            request.post("bookwithuser/deleteRecord", form3).then(res => {
                console.log(res)
                if (res.code == 0) {
                    ElMessage.success("归还成功")
                    // 2. 同步更新借阅记录状态为已归还
                    const returnBookRequest = {
                        status: '1', // 设置为已归还状态
                        returnTime: new Date().toISOString().slice(0, 19).replace('T', ' '), // 设置当前时间为归还时间
                        // 只包含RequestLendRecord.java中定义的字段
                    }
                    // 调用借阅记录更新接口
                    request.put(`LendRecord/${row.isbn}`, returnBookRequest).then(lendRes => {
                        console.log('借阅记录状态更新：', lendRes)
                        if (lendRes.code === 0) {
                            ElMessage.success("借阅记录状态更新成功")
                        } else {
                            ElMessage.error("借阅记录状态更新失败：" + lendRes.msg)
                        }
                    }).catch(error => {
                        console.error('更新借阅记录状态失败：', error)
                        ElMessage.error("更新借阅记录状态失败，请检查网络连接")
                    }).finally(() => {
                        // 3. 更新图书状态为已归还（status=1）
                        request.put(`book/status?isbn=${row.isbn}&status=1`).then(bookRes => {
                            console.log('图书状态更新：', bookRes)
                            if (bookRes.code === 0) {
                                ElMessage.success("图书状态更新成功，已设置为已归还")
                            } else {
                                ElMessage.error("图书状态更新失败：" + bookRes.msg)
                            }
                        }).catch(error => {
                            console.error('更新图书状态失败：', error)
                            ElMessage.error("更新图书状态失败，请检查网络连接")
                        }).finally(() => {
                            // 无论成功失败，都重新加载数据
                            this.load()
                        })
                    })
                } else {
                    ElMessage.error(res.msg)
                    this.load()
                }
            })
        },
        save() {
            //ES6语法
            //地址,但是？IP与端口？+请求参数
            // this.form?这是自动保存在form中的，虽然显示时没有使用，但是这个对象中是有它的
            request.post("bookwithuser", this.form).then(res => {
                console.log(res)
                if (res.code == 0) {
                    ElMessage({
                        message: '修改信息成功',
                        type: 'success',
                    })
                }
                else {
                    ElMessage.error(res.msg)
                }
                this.load()
                this.dialogVisible2 = false
            })
        },

        handleEdit(row) {
            this.form = JSON.parse(JSON.stringify(row))
            this.dialogVisible2 = true
        },
        handleSizeChange(pageSize) {
            this.pageSize = pageSize
            this.load()
        },
        handleCurrentChange(pageNum) {
            this.pageNum = pageNum
            this.load()
        },

    },
    data() {
        return {
            form: {},
            form2: {},
            form3: {},
            dialogVisible: false,
            dialogVisible2: false,
            search1: '',
            search2: '',
            search3: '',
            total: 10,
            currentPage: 1,
            pageSize: 10,
            tableData: [],
            user: {},
            forms: [],
        }
    },
}
</script>

<style scoped>
.no-permission {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 400px;
}
</style>