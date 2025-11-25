<template>
  <div class="lendrecord-container">
    <!-- 页面标题 -->
    <div class="page-header">
      <h1 class="page-title">📚 借阅记录管理</h1>
      <p class="page-subtitle">管理图书借阅和归还记录</p>
    </div>

    <!-- 搜索卡片 -->
    <el-card class="search-card">
      <template #header>
        <div class="card-header">
          <el-icon>
            <Search />
          </el-icon>
          <span>记录搜索</span>
        </div>
      </template>
      <el-form inline="true" size="default">
        <el-row :gutter="20">
          <el-col :xs="24" :sm="12" :md="8">
            <el-form-item label="图书编号">
              <el-input v-model="search1" placeholder="请输入图书编号" clearable>
                <template #prefix><el-icon class="el-input__icon">
                    <Search />
                  </el-icon></template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="12" :md="8">
            <el-form-item label="图书名称">
              <el-input v-model="search2" placeholder="请输入图书名称" clearable>
                <template #prefix><el-icon class="el-input__icon">
                    <Search />
                  </el-icon></template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="12" :md="8">
            <el-form-item label="读者编号">
              <el-input v-model="search3" placeholder="请输入读者编号" clearable>
                <template #prefix><el-icon class="el-input__icon">
                    <Search />
                  </el-icon></template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row justify="end">
          <el-col :span="24" style="text-align: right;">
            <el-button type="primary" @click="load" size="default">
              <el-icon>
                <Search />
              </el-icon>
              查询
            </el-button>
            <el-button size="default" @click="clear">
              <el-icon>
                <Refresh />
              </el-icon>
              重置
            </el-button>
          </el-col>
        </el-row>
      </el-form>
    </el-card>

    <!-- 操作按钮卡片 -->
    <el-card class="action-card" v-if="user.role == 1">
      <template #header>
        <div class="card-header">
          <el-icon>
            <Operation />
          </el-icon>
          <span>批量操作</span>
        </div>
      </template>
      <el-popconfirm title="确认删除选中的记录?" @confirm="deleteBatch">
        <template #reference>
          <el-button type="danger" size="default">
            <el-icon>
              <Delete />
            </el-icon>
            批量删除
          </el-button>
        </template>
      </el-popconfirm>
    </el-card>

    <!-- 数据表格卡片 -->
    <el-card class="table-card">
      <template #header>
        <div class="card-header">
          <el-icon>
            <Document />
          </el-icon>
          <span>借阅记录列表</span>
          <el-tag type="info" style="margin-left: auto;">共 {{ total }} 条记录</el-tag>
        </div>
      </template>

      <el-table :data="tableData" stripe border @selection-change="handleSelectionChange" class="modern-table"
        v-loading="loading" element-loading-text="正在加载借阅记录..." element-loading-background="rgba(0, 0, 0, 0.1)">

        <el-table-column v-if="user.role == 1" type="selection" width="55" />
        <el-table-column prop="isbn" label="图书编号" sortable width="120">
          <template #default="scope">
            <el-tag type="primary" size="small">{{ scope.row.isbn }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="bookname" label="图书名称" min-width="200" show-overflow-tooltip />
        <el-table-column prop="readerId" label="读者编号" sortable width="120">
          <template #default="scope">
            <el-tag type="info" size="small">{{ scope.row.readerId }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="lendTime" label="借阅时间" sortable width="180">
          <template #default="scope">
            <div style="color: #e6a23c;">
              <el-icon>
                <Clock />
              </el-icon>
              {{ scope.row.lendTime }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="returnTime" label="归还时间" sortable width="180">
          <template #default="scope">
            <div :style="{ color: scope.row.status == 0 ? '#f56c6c' : '#67c23a' }">
              <el-icon>
                <Timer />
              </el-icon>
              {{ scope.row.returnTime || '未归还' }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="状态" width="100">
          <template #default="scope">
            <el-tag v-if="scope.row.status == 0" type="warning" size="large">
              <el-icon>
                <Warning />
              </el-icon>
              未归还
            </el-tag>
            <el-tag v-else type="success" size="large">
              <el-icon>
                <CircleCheck />
              </el-icon>
              已归还
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column v-if="user.role === 1" label="操作" width="150">
          <template #default="scope">
            <div class="action-buttons">
              <el-button size="small" @click="handleEdit(scope.row)" type="primary" plain>
                <el-icon>
                  <Edit />
                </el-icon>
                编辑
              </el-button>
              <el-popconfirm title="确认删除此记录?" @confirm="handleDelete(scope.row)">
                <template #reference>
                  <el-button type="danger" size="small" plain>
                    <el-icon>
                      <Delete />
                    </el-icon>
                    删除
                  </el-button>
                </template>
              </el-popconfirm>
            </div>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination-container">
        <el-pagination v-model:currentPage="currentPage" :page-sizes="[5, 10, 20, 50]" :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
          @current-change="handleCurrentChange" background />
      </div>
    </el-card>

    <!-- 编辑对话框 -->
    <el-dialog v-model="dialogVisible" title="修改借阅记录" width="500px" class="modern-dialog">
      <el-form :model="form" label-width="120px" label-position="left">
        <el-form-item label="借阅时间">
          <el-date-picker v-model="form.lendTime" type="datetime" value-format="YYYY-MM-DD HH:mm:ss"
            placeholder="选择借阅时间" style="width: 100%;" />
        </el-form-item>
        <el-form-item label="归还时间">
          <el-date-picker v-model="form.returnTime" type="datetime" value-format="YYYY-MM-DD HH:mm:ss"
            placeholder="选择归还时间" style="width: 100%;" />
        </el-form-item>
        <el-form-item label="借阅状态">
          <el-radio-group v-model="form.status">
            <el-radio label="0">
              <el-icon>
                <Warning />
              </el-icon>
              未归还
            </el-radio>
            <el-radio label="1">
              <el-icon>
                <CircleCheck />
              </el-icon>
              已归还
            </el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="save(form.isbn)">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>

import request from "../utils/request";
import { ElMessage } from "element-plus";
import { defineComponent, reactive, toRefs } from 'vue'
import router from "@/router";
import { Search, Operation, Document, Clock, Timer, Warning, CircleCheck, Edit, Delete, Refresh } from '@element-plus/icons-vue'

export default defineComponent({
  components: {
    Search,
    Operation,
    Document,
    Clock,
    Timer,
    Warning,
    CircleCheck,
    Edit,
    Delete,
    Refresh
  },

  created() {
    let userJson = sessionStorage.getItem("user")
    if (!userJson) {
      router.push("/login")
    }
    this.load()
    let userStr = sessionStorage.getItem("user") || "{}"
    this.user = JSON.parse(userStr)
  },
  name: 'LendRecord',
  data() {
    return {
      form: {},
      search1: '',
      search2: '',
      search3: '',
      total: 10,
      currentPage: 1,
      pageSize: 10,
      tableData: [],
      user: {},
      dialogVisible: false,
      dialogVisible2: false,
      loading: false,
      forms: []
    }
  },
  methods: {
    handleSelectionChange(val) {
      this.forms = val
    },
    deleteBatch() {
      if (!this.forms.length) {
        ElMessage.warning("请选择数据！")
        return
      }
      request.post("/LendRecord/deleteRecords", this.forms).then(res => {
        if (res.code === '0') {
          ElMessage.success("批量删除成功")
          this.load()
        }
        else {
          ElMessage.error(res.msg)
        }
      })
    },
    load() {
      this.loading = true
      request.get("/LendRecord", {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize,
          search1: this.search1,
          search2: this.search2,
          search3: this.search3
        }
      }).then(res => {
        console.log(res)
        this.tableData = res.data.records
        this.total = res.data.total
        this.loading = false
      }).catch(() => {
        this.loading = false
      })
    },
    save(isbn) {
      if (this.form.isbn) {
        request.post("/LendRecord" + isbn, this.form).then(res => {
          console.log(res)
          if (res.code == 0) {
            ElMessage({
              message: '新增成功',
              type: 'success',
            })
          } else {
            ElMessage.error(res.msg)
          }
          this.load()
          this.dialogVisible = false
        })
      }
      else {
        request.put("/LendRecord/" + isbn, this.form).then(res => {
          console.log(res)
          if (res.code == 0) {
            ElMessage({
              message: '更新成功',
              type: 'success',
            })
          } else {
            ElMessage.error(res.msg)
          }
          this.load()
          this.dialogVisible2 = false
        })
      }
    },
    clear() {
      this.search1 = ""
      this.search2 = ""
      this.search3 = ""
      this.load()
    },
    handleEdit(row) {
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible = true
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.load()
    },
    handleCurrentChange(pageNum) {
      this.currentPage = pageNum
      this.load()
    },
    handleDelete(row) {
      const form3 = JSON.parse(JSON.stringify(row))
      request.post("LendRecord/deleteRecord", form3).then(res => {
        console.log(res)
        if (res.code == 0) {
          ElMessage.success("删除成功")
        }
        else
          ElMessage.error(res.msg)
        this.load()
      })
    },
    add() {
      this.dialogVisible2 = true
      this.form = {}
    }
  },

  setup() {
    const state = reactive({
      shortcuts: [
        {
          text: 'Today',
          value: new Date(),
        },
        {
          text: 'Yesterday',
          value: () => {
            const date = new Date()
            date.setTime(date.getTime() - 3600 * 1000 * 24)
            return date
          },
        },
        {
          text: 'A week ago',
          value: () => {
            const date = new Date()
            date.setTime(date.getTime() - 3600 * 1000 * 24 * 7)
            return date
          },
        },
      ],
      value1: '',
      value2: '',
      value3: '',
      defaultTime: new Date(2000, 1, 1, 12, 0, 0), // '12:00:00'
    })

    return toRefs(state)
  }
})
</script>


<style scoped>
/* 主容器样式 */
.lendrecord-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  overflow-x: auto;
}

/* 页面标题 */
.page-header {
  text-align: center;
  margin-bottom: 30px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.page-title {
  font-size: 2.5rem;
  font-weight: bold;
  background: linear-gradient(45deg, #fff, #f0f0f0);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  margin: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.page-subtitle {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1.1rem;
  margin-top: 10px;
}

/* 卡片通用样式 */
:deep(.el-card) {
  border-radius: 20px;
  border: none;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  transition: all 0.3s ease;
}

:deep(.el-card:hover) {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

/* 卡片头部 */
.card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: bold;
  color: #333;
  font-size: 1.1rem;
}

.card-header .el-icon {
  font-size: 1.2rem;
  color: #667eea;
}

/* 搜索表单样式 */
:deep(.el-form-item__label) {
  font-weight: 500;
  color: #555;
}

:deep(.el-input__wrapper) {
  border-radius: 12px;
  border: 1px solid #e0e0e0;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

:deep(.el-input__wrapper:hover) {
  border-color: #667eea;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.15);
}

:deep(.el-input__wrapper.is-focus) {
  border-color: #667eea;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.2);
}

/* 按钮样式 */
:deep(.el-button) {
  border-radius: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

:deep(.el-button:hover) {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

:deep(.el-button--primary) {
  background: linear-gradient(45deg, #667eea, #764ba2);
  border: none;
}

:deep(.el-button--danger) {
  background: linear-gradient(45deg, #ff6b6b, #ee5a24);
  border: none;
}

:deep(.el-button--primary.is-plain) {
  border: 1px solid #667eea;
  color: #667eea;
}

:deep(.el-button--danger.is-plain) {
  border: 1px solid #ff6b6b;
  color: #ff6b6b;
}

/* 表格样式 */
.modern-table {
  border-radius: 12px;
  overflow: hidden;
}

:deep(.el-table__header-wrapper) {
  border-radius: 12px 12px 0 0;
}

:deep(.el-table__header) {
  background: linear-gradient(45deg, #667eea, #764ba2);
}

:deep(.el-table th) {
  background: transparent;
  color: white;
  font-weight: 600;
  border: none;
  padding: 12px 8px;
}

:deep(.el-table th .cell) {
  color: white;
}

:deep(.el-table--striped .el-table__body tr.el-table__row--striped td) {
  background: rgba(102, 126, 234, 0.05);
}

:deep(.el-table__row:hover td) {
  background: rgba(102, 126, 234, 0.1) !important;
  transition: all 0.3s ease;
}

:deep(.el-table td) {
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  padding: 12px 8px;
}

/* 状态标签样式 */
:deep(.el-tag) {
  border-radius: 20px;
  font-weight: 500;
  padding: 6px 12px;
}

:deep(.el-tag--warning) {
  background: linear-gradient(45deg, #e6a23c, #f56c6c);
  color: white;
  border: none;
}

:deep(.el-tag--success) {
  background: linear-gradient(45deg, #67c23a, #529b2f);
  color: white;
  border: none;
}

:deep(.el-tag--primary) {
  background: linear-gradient(45deg, #409eff, #3b8beb);
  color: white;
  border: none;
}

:deep(.el-tag--info) {
  background: linear-gradient(45deg, #909399, #73767a);
  color: white;
  border: none;
}

/* 操作按钮组 */
.action-buttons {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.action-buttons .el-button {
  margin: 0;
}

/* 分页样式 */
.pagination-container {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

:deep(.el-pagination) {
  padding: 20px 0;
}

:deep(.el-pagination .btn-prev),
:deep(.el-pagination .btn-next),
:deep(.el-pagination .el-pager li) {
  border-radius: 8px;
  margin: 0 4px;
  transition: all 0.3s ease;
}

:deep(.el-pagination .el-pager li) {
  background: rgba(255, 255, 255, 0.8);
  border: 1px solid #e0e0e0;
}

:deep(.el-pagination .el-pager li:hover) {
  background: rgba(102, 126, 234, 0.1);
  border-color: #667eea;
}

:deep(.el-pagination .el-pager li.is-active) {
  background: linear-gradient(45deg, #667eea, #764ba2);
  border-color: #667eea;
  color: white;
}

/* 对话框样式 */
.modern-dialog {
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
}

:deep(.el-dialog__header) {
  background: linear-gradient(45deg, #667eea, #764ba2);
  border-radius: 20px 20px 0 0;
  margin: 0;
  padding: 20px;
}

:deep(.el-dialog__title) {
  color: white;
  font-weight: 600;
  font-size: 1.2rem;
}

:deep(.el-dialog__body) {
  padding: 30px 20px;
}

:deep(.el-dialog__footer) {
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  padding: 15px 20px;
}

/* 单选框组样式 */
:deep(.el-radio-group) {
  display: flex;
  gap: 20px;
}

:deep(.el-radio) {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* 日期选择器样式 */
:deep(.el-date-editor) {
  border-radius: 12px;
}

:deep(.el-date-editor .el-input__wrapper) {
  border-radius: 12px;
}

/* 加载动画 */
:deep(.el-loading-spinner) {
  top: 50%;
  transform: translateY(-50%);
}

:deep(.el-loading-spinner .el-loading-text) {
  color: #667eea;
  font-size: 1.1rem;
  font-weight: 500;
}

:deep(.el-loading-spinner .path) {
  stroke: #667eea;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .lendrecord-container {
    padding: 10px;
  }

  .page-title {
    font-size: 2rem;
  }

  .page-subtitle {
    font-size: 1rem;
  }

  .action-buttons {
    flex-direction: column;
  }

  .action-buttons .el-button {
    width: 100%;
  }
}

/* 滚动条样式 */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: linear-gradient(45deg, #667eea, #764ba2);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(45deg, #764ba2, #667eea);
}
</style>