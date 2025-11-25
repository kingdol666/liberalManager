<template>
  <div class="book-container">
    <!-- 页面标题 -->
    <div class="page-header">
      <h1 class="page-title">📚 图书管理</h1>
      <p class="page-subtitle">管理图书馆藏书，追踪借阅状态</p>
    </div>

    <!-- 搜索区域 -->
    <el-card class="search-card" shadow="hover">
      <template #header>
        <div class="card-header">
          <span class="card-title">🔍 图书搜索</span>
        </div>
      </template>
      <el-form :inline="true" size="small" class="search-form">
        <el-form-item label="图书编号" class="form-item">
          <el-input v-model="search1" placeholder="请输入图书编号" clearable class="modern-input">
            <template #prefix><el-icon class="el-input__icon">
                <search />
              </el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="图书名称" class="form-item">
          <el-input v-model="search2" placeholder="请输入图书名称" clearable class="modern-input">
            <template #prefix><el-icon class="el-input__icon">
                <search />
              </el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="作者" class="form-item">
          <el-input v-model="search3" placeholder="请输入作者" clearable class="modern-input">
            <template #prefix><el-icon class="el-input__icon">
                <search />
              </el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item class="form-item">
          <el-button type="primary" class="modern-button" @click="load">
            <el-icon>
              <Search />
            </el-icon>查询
          </el-button>
        </el-form-item>
        <el-form-item class="form-item">
          <el-button class="modern-button reset" @click="clear">
            <el-icon>
              <Refresh />
            </el-icon>重置
          </el-button>
        </el-form-item>
        <el-form-item class="form-item" v-if="numOfOutDataBook != 0">
          <el-popconfirm confirm-button-text="查看" cancel-button-text="取消" :icon="InfoFilled" icon-color="red"
            title="您有图书已逾期，请尽快归还" @confirm="toLook">
            <template #reference>
              <el-button type="warning" class="modern-button warning">
                <el-icon>
                  <Warning />
                </el-icon>逾期通知
              </el-button>
            </template>
          </el-popconfirm>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 操作按钮区域 -->
    <el-card class="action-card" shadow="hover">
      <div class="action-buttons">
        <el-button type="primary" class="modern-button primary" @click="add" v-if="user.role == 1">
          <el-icon>
            <Plus />
          </el-icon>上架图书
        </el-button>
        <el-popconfirm title="确认下架选中的图书?" @confirm="deleteBatch" v-if="user.role == 1">
          <template #reference>
            <el-button type="danger" class="modern-button danger">
              <el-icon>
                <Delete />
              </el-icon>批量下架
            </el-button>
          </template>
        </el-popconfirm>
      </div>
    </el-card>

    <!-- 数据表格 -->
    <el-card class="table-card" shadow="hover">
      <template #header>
        <div class="card-header">
          <span class="card-title">📖 图书列表</span>
          <span class="card-subtitle">共 {{ total }} 本图书</span>
        </div>
      </template>
      <el-table :data="tableData" stripe border class="modern-table" @selection-change="handleSelectionChange"
        v-loading="loading" element-loading-text="正在加载图书数据..." element-loading-spinner="el-icon-loading">
        <el-table-column v-if="user.role == 1" type="selection" width="55" align="center" />
        <el-table-column prop="isbn" label="图书编号" sortable align="center" width="120">
          <template #default="scope">
            <span class="isbn-text">{{ scope.row.isbn }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="name" label="图书名称" align="center" min-width="200">
          <template #default="scope">
            <div class="book-name">
              <el-icon>
                <Notebook />
              </el-icon>
              <span>{{ scope.row.name }}</span>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="price" label="价格" sortable align="center" width="100">
          <template #default="scope">
            <span class="price-text">¥{{ scope.row.price }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="author" label="作者" align="center" width="120" />
        <el-table-column prop="publisher" label="出版社" align="center" width="150" />
        <el-table-column prop="createTime" label="出版时间" sortable align="center" width="120">
          <template #default="scope">
            <span class="date-text">{{ scope.row.createTime }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="borrownum" label="借阅次数" sortable align="center" width="100">
          <template #default="scope">
            <el-tag size="small" type="info">{{ scope.row.borrownum }} 次</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="状态" align="center" width="100">
          <template v-slot="scope">
            <el-tag v-if="scope.row.status == 0" type="warning" class="status-tag borrowed">
              <el-icon>
                <Clock />
              </el-icon>已借阅
            </el-tag>
            <el-tag v-else type="success" class="status-tag available">
              <el-icon>
                <Check />
              </el-icon>未借阅
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column fixed="right" label="操作" align="center" width="200">
          <template v-slot="scope">
            <div class="action-group">
              <el-button size="small" @click="handleEdit(scope.row)" v-if="user.role == 1" class="action-btn edit">
                <el-icon>
                  <Edit />
                </el-icon>修改
              </el-button>
              <el-popconfirm title="确认下架此图书?" @confirm="handleDelete(scope.row.id)" v-if="user.role == 1">
                <template #reference>
                  <el-button type="danger" size="small" class="action-btn delete">
                    <el-icon>
                      <Delete />
                    </el-icon>下架
                  </el-button>
                </template>
              </el-popconfirm>
              <el-button size="small"
                @click="handlelend(scope.row.id, scope.row.isbn, scope.row.name, scope.row.borrownum)"
                v-if="user.role == 2" :disabled="scope.row.status == 0" class="action-btn lend">
                <el-icon>
                  <Reading />
                </el-icon>借阅
              </el-button>
              <el-popconfirm title="确认归还此图书?" @confirm="handlereturn(scope.row.id, scope.row.isbn, scope.row.borrownum)"
                v-if="user.role == 2" :disabled="scope.row.status == 1">
                <template #reference>
                  <el-button type="danger" size="small"
                    :disabled="(this.isbnArray.indexOf(scope.row.isbn)) == -1 || scope.row.status == 1"
                    class="action-btn return">
                    <el-icon>
                      <RefreshLeft />
                    </el-icon>还书
                  </el-button>
                </template>
              </el-popconfirm>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分页 -->
    <div class="pagination-container">
      <el-pagination v-model:currentPage="currentPage" :page-sizes="[5, 10, 12, 20, 50]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
        @current-change="handleCurrentChange" class="modern-pagination" />
    </div>

    <!-- 逾期通知对话框 -->
    <el-dialog v-model="dialogVisible3" v-if="numOfOutDataBook != 0" title="📅 逾期详情" width="50%" class="modern-dialog"
      :before-close="handleClose">
      <el-table :data="outDateBook" style="width: 100%" class="dialog-table">
        <el-table-column prop="isbn" label="图书编号" width="120" />
        <el-table-column prop="bookName" label="书名" min-width="200" />
        <el-table-column prop="lendtime" label="借阅日期" width="120" />
        <el-table-column prop="deadtime" label="截止日期" width="120" />
      </el-table>
      <template #footer>
        <span class="dialog-footer">
          <el-button type="primary" @click="dialogVisible3 = false" class="modern-button">
            确认
          </el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 上架图书对话框 -->
    <el-dialog v-model="dialogVisible" title="📚 上架图书" width="35%" class="modern-dialog">
      <el-form :model="form" label-width="100px" class="modern-form">
        <el-form-item label="图书编号" class="form-item">
          <el-input v-model="form.isbn" class="modern-input" placeholder="请输入图书编号" />
        </el-form-item>
        <el-form-item label="图书名称" class="form-item">
          <el-input v-model="form.name" class="modern-input" placeholder="请输入图书名称" />
        </el-form-item>
        <el-form-item label="价格" class="form-item">
          <el-input v-model="form.price" class="modern-input" placeholder="请输入价格" />
        </el-form-item>
        <el-form-item label="作者" class="form-item">
          <el-input v-model="form.author" class="modern-input" placeholder="请输入作者姓名" />
        </el-form-item>
        <el-form-item label="出版社" class="form-item">
          <el-input v-model="form.publisher" class="modern-input" placeholder="请输入出版社" />
        </el-form-item>
        <el-form-item label="出版时间" class="form-item">
          <el-date-picker value-format="YYYY-MM-DD" type="date" class="modern-date-picker" clearable
            v-model="form.createTime" placeholder="请选择出版时间">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false" class="modern-button">取 消</el-button>
          <el-button type="primary" @click="save" class="modern-button primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 修改图书对话框 -->
    <el-dialog v-model="dialogVisible2" title="✏️ 修改图书信息" width="35%" class="modern-dialog">
      <el-form :model="form" label-width="100px" class="modern-form">
        <el-form-item label="图书编号" class="form-item">
          <el-input v-model="form.isbn" class="modern-input" placeholder="请输入图书编号" />
        </el-form-item>
        <el-form-item label="图书名称" class="form-item">
          <el-input v-model="form.name" class="modern-input" placeholder="请输入图书名称" />
        </el-form-item>
        <el-form-item label="价格" class="form-item">
          <el-input v-model="form.price" class="modern-input" placeholder="请输入价格" />
        </el-form-item>
        <el-form-item label="作者" class="form-item">
          <el-input v-model="form.author" class="modern-input" placeholder="请输入作者姓名" />
        </el-form-item>
        <el-form-item label="出版社" class="form-item">
          <el-input v-model="form.publisher" class="modern-input" placeholder="请输入出版社" />
        </el-form-item>
        <el-form-item label="出版时间" class="form-item">
          <el-date-picker value-format="YYYY-MM-DD" type="date" class="modern-date-picker" clearable
            v-model="form.createTime" placeholder="请选择出版时间">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible2 = false" class="modern-button">取 消</el-button>
          <el-button type="primary" @click="save" class="modern-button primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>
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
    let userJson = sessionStorage.getItem("user")
    if (!userJson) {
      router.push("/login")
    }
    let userStr = sessionStorage.getItem("user") || "{}"
    this.user = JSON.parse(userStr)
    let user = JSON.parse(sessionStorage.getItem("user"))
    this.phone = user.phone
    this.load()
  },
  name: 'Book',
  methods: {
    // (this.isbnArray.indexOf(scope.row.isbn)) == -1
    handleSelectionChange(val) {
      this.ids = val.map(v => v.id)
    },
    deleteBatch() {
      if (!this.ids.length) {
        ElMessage.warning("请选择数据！")
        return
      }
      //  一个小优化，直接发送这个数组，而不是一个一个的提交下架
      request.post("/book/deleteBatch", this.ids).then(res => {
        if (res.code === '0') {
          ElMessage.success("批量下架成功")
          this.load()
        }
        else {
          ElMessage.error(res.msg)
        }
      })
    },
    load() {
      this.numOfOutDataBook = 0;
      this.outDateBook = [];
      request.get("/book", {
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
      //
      if (this.user.role == 2) {
        request.get("/bookwithuser", {
          params: {
            pageNum: "1",
            pageSize: this.total,
            search1: "",
            search2: "",
            search3: this.user.id,
          }
        }).then(res => {
          console.log(res)
          this.bookData = res.data.records
          this.number = this.bookData.length;
          var nowDate = new Date();
          for (let i = 0; i < this.number; i++) {
            this.isbnArray[i] = this.bookData[i].isbn;
            let dDate = new Date(this.bookData[i].deadtime);
            if (dDate < nowDate) {
              this.outDateBook[this.numOfOutDataBook] = {
                isbn: this.bookData[i].isbn,
                bookName: this.bookData[i].bookName,
                deadtime: this.bookData[i].deadtime,
                lendtime: this.bookData[i].lendtime,
              };
              this.numOfOutDataBook = this.numOfOutDataBook + 1;
            }
          }
          console.log("in load():" + this.numOfOutDataBook);
        })
      }
      request.get("/user/alow/" + this.user.id).then(res => {
        if (res.code == 0) {
          this.flag = true
        }
        else {
          this.flag = false
        }
      })
      //判断是否具有借阅权力
    },
    clear() {
      this.search1 = ""
      this.search2 = ""
      this.search3 = ""
      this.load()
    },

    handleDelete(id) {
      request.delete("book/" + id).then(res => {
        console.log(res)
        if (res.code == 0) {
          ElMessage.success("下架成功")
        }
        else
          ElMessage.error(res.msg)
        this.load()
      })
    },
    handlereturn(id, isbn, bn) {
      this.form.status = "1"
      this.form.id = id
      request.put("/book", this.form).then(res => {
        console.log(res)
        if (res.code == 0) {
          ElMessage({
            message: '还书成功',
            type: 'success',
          })
        }
        else {
          ElMessage.error(res.msg)
        }
        //
        this.form3.isbn = isbn
        this.form3.readerId = this.user.id
        let endDate = moment(new Date()).format("yyyy-MM-DD HH:mm:ss")
        this.form3.returnTime = endDate
        this.form3.status = "1"
        console.log(bn)
        this.form3.borrownum = bn
        request.put("/LendRecord1/", this.form3).then(res => {
          console.log(res)
          let form3 = {};
          form3.isbn = isbn;
          form3.bookName = name;
          form3.nickName = this.user.username;
          form3.id = this.user.id;
          form3.lendtime = endDate;
          form3.deadtime = endDate;
          form3.prolong = 1;
          request.post("/bookwithuser/deleteRecord", form3).then(res => {
            console.log(res)
            this.load()
          })

        })
        //
      })
    },
    handlelend(id, isbn, name, bn) {

      if (this.phone == null) {
        ElMessage.error("借阅失败! 请先将个人信息补充完整")
        this.$router.push("/person")//跳转个人信息界面
        return;
      }

      if (this.number == 5) {
        ElMessage.warning("您不能再借阅更多的书籍了")
        return;
      }
      if (this.numOfOutDataBook != 0) {
        ElMessage.warning("在您归还逾期书籍前不能再借阅书籍")
        return;
      }

      if (this.flag == false) {
        ElMessage({
          message: '您没有借阅权限,管理员审核通过后授权',
          type: 'error',
        })
        return;
      }

      this.form.status = "0"
      this.form.id = id
      this.form.borrownum = bn + 1
      console.log(bn)
      request.put("/book", this.form).then(res => {
        console.log(res)
        if (res.code == 0) {
          ElMessage({
            message: '借阅成功',
            type: 'success',
          })
        }
        else {
          ElMessage.error(res.msg)
        }
      })

      this.form2.status = "0"
      this.form2.isbn = isbn
      this.form2.bookname = name
      this.form2.readerId = this.user.id
      this.form2.borrownum = bn + 1
      console.log(this.form2.borrownum)
      console.log(this.user)
      let startDate = moment(new Date()).format("yyyy-MM-DD HH:mm:ss");
      this.form2.lendTime = startDate
      console.log(this.user)
      request.post("/LendRecord", this.form2).then(res => {
        console.log(res)
        this.load();

      })
      let form3 = {};
      form3.isbn = isbn;
      form3.bookName = name;
      form3.nickName = this.user.username;
      form3.id = this.user.id;
      form3.lendtime = startDate;
      let nowDate = new Date(startDate);
      nowDate.setDate(nowDate.getDate() + 30);
      form3.deadtime = moment(nowDate).format("yyyy-MM-DD HH:mm:ss");
      form3.prolong = 1;
      request.post("/bookwithuser/insertNew", form3).then(res => {
        console.log(res)
        this.load()
      })
    },
    add() {
      this.dialogVisible = true
      this.form = {}
    },
    save() {
      //ES6语法
      //地址,但是？IP与端口？+请求参数
      // this.form?这是自动保存在form中的，虽然显示时没有使用，但是这个对象中是有它的
      if (this.form.id) {
        request.put("/book", this.form).then(res => {
          console.log(res)
          if (res.code == 0) {
            ElMessage({
              message: '修改书籍信息成功',
              type: 'success',
            })
          }
          else {
            ElMessage.error(res.msg)
          }

          this.load()
          this.dialogVisible2 = false
        })
      }
      else {
        this.form.borrownum = 0
        this.form.status = 1
        request.post("/book", this.form).then(res => {
          console.log(res)
          if (res.code == 0) {
            ElMessage.success('上架书籍成功')
          }
          else {
            ElMessage.error(res.msg)
          }
          this.load()
          this.dialogVisible = false
        })
      }

    },
    // formatter(row) {:formatter="formatter"
    //   return row.address
    // },

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
    toLook() {
      this.dialogVisible3 = true;
    },
  },
  data() {
    return {
      phone: '',
      flag: '',
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
      number: 0,
      bookData: [],
      isbnArray: [],
      outDateBook: [],
      numOfOutDataBook: 0,
      dialogVisible3: true,
    }
  },
}
</script>

<!-- 在template标签之后添加style标签 -->

<style scoped>
.book-container {
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
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
}

.page-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  background: linear-gradient(45deg, #fff, #f0f8ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.page-subtitle {
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.8);
  margin: 10px 0 0 0;
  font-weight: 300;
}

/* 卡片样式 */
.search-card,
.action-card,
.table-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  margin-bottom: 20px;
  border: none;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.search-card:hover,
.action-card:hover,
.table-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
}

.card-title {
  font-size: 1.3rem;
  font-weight: 600;
  color: #2c3e50;
  display: flex;
  align-items: center;
  gap: 8px;
}

.card-subtitle {
  font-size: 0.9rem;
  color: #7f8c8d;
  font-weight: 400;
}

/* 搜索表单 */
.search-form {
  padding: 20px 0;
}

.form-item {
  margin-right: 20px;
  margin-bottom: 15px;
}

.form-item :deep(.el-form-item__label) {
  font-weight: 600;
  color: #34495e;
  font-size: 0.95rem;
}

/* 现代输入框 */
.modern-input {
  border-radius: 12px;
  border: 2px solid #e0e6ed;
  transition: all 0.3s ease;
  background: #ffffff;
}

.modern-input:hover {
  border-color: #409eff;
  box-shadow: 0 0 0 4px rgba(64, 158, 255, 0.1);
}

.modern-input:focus-within {
  border-color: #409eff;
  box-shadow: 0 0 0 6px rgba(64, 158, 255, 0.15);
}

/* 现代按钮 */
.modern-button {
  border-radius: 12px;
  font-weight: 600;
  padding: 10px 20px;
  transition: all 0.3s ease;
  border: none;
  display: flex;
  align-items: center;
  gap: 6px;
}

.modern-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.modern-button.primary {
  background: linear-gradient(45deg, #409eff, #66b1ff);
  color: white;
}

.modern-button.primary:hover {
  background: linear-gradient(45deg, #66b1ff, #409eff);
}

.modern-button.reset {
  background: linear-gradient(45deg, #909399, #b4b6ba);
  color: white;
}

.modern-button.warning {
  background: linear-gradient(45deg, #e6a23c, #f3d19e);
  color: white;
}

.modern-button.danger {
  background: linear-gradient(45deg, #f56c6c, #fab6b6);
  color: white;
}

/* 操作按钮区域 */
.action-buttons {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

/* 现代表格 */
.modern-table {
  border-radius: 12px;
  overflow: hidden;
  border: none;
}

.modern-table :deep(.el-table__header-wrapper) {
  background: linear-gradient(45deg, #f8f9fa, #e9ecef);
}

.modern-table :deep(.el-table__header) {
  background: transparent;
}

.modern-table :deep(.el-table__row) {
  transition: all 0.3s ease;
}

.modern-table :deep(.el-table__row:hover) {
  background: linear-gradient(45deg, #f0f8ff, #e6f7ff);
  transform: scale(1.01);
}

/* 表格内容样式 */
.isbn-text {
  font-weight: 600;
  color: #2c3e50;
  font-family: 'Courier New', monospace;
}

.book-name {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
  color: #34495e;
}

.price-text {
  font-weight: 600;
  color: #27ae60;
  font-size: 1.1rem;
}

.date-text {
  color: #7f8c8d;
  font-size: 0.9rem;
}

/* 状态标签 */
.status-tag {
  border-radius: 20px;
  font-weight: 600;
  padding: 6px 12px;
  display: flex;
  align-items: center;
  gap: 4px;
  justify-content: center;
}

.status-tag.borrowed {
  background: linear-gradient(45deg, #e6a23c, #f3d19e);
  border: none;
}

.status-tag.available {
  background: linear-gradient(45deg, #67c23a, #b3e19d);
  border: none;
}

/* 操作按钮组 */
.action-group {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: center;
}

.action-btn {
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.3s ease;
  padding: 6px 12px;
  font-size: 0.85rem;
}

.action-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.action-btn.edit {
  background: linear-gradient(45deg, #409eff, #66b1ff);
  border: none;
  color: white;
}

.action-btn.delete {
  background: linear-gradient(45deg, #f56c6c, #fab6b6);
  border: none;
  color: white;
}

.action-btn.lend {
  background: linear-gradient(45deg, #67c23a, #b3e19d);
  border: none;
  color: white;
}

.action-btn.return {
  background: linear-gradient(45deg, #e6a23c, #f3d19e);
  border: none;
  color: white;
}

/* 分页 */
.pagination-container {
  display: flex;
  justify-content: center;
  margin-top: 30px;
}

.modern-pagination {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 20px;
  padding: 10px 20px;
  backdrop-filter: blur(10px);
  border: none;
}

.modern-pagination :deep(.el-pagination__total),
.modern-pagination :deep(.el-pagination__sizes),
.modern-pagination :deep(.el-pagination__jump) {
  color: #34495e;
  font-weight: 500;
}

/* 对话框 */
.modern-dialog {
  border-radius: 20px;
  overflow: hidden;
}

.modern-dialog :deep(.el-dialog__header) {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  padding: 20px;
  margin: 0;
}

.modern-dialog :deep(.el-dialog__title) {
  color: white;
  font-weight: 600;
  font-size: 1.3rem;
}

.modern-dialog :deep(.el-dialog__body) {
  padding: 30px;
  background: #f8f9fa;
}

.modern-dialog :deep(.el-dialog__footer) {
  background: #f8f9fa;
  padding: 20px 30px;
  border-top: 1px solid #e9ecef;
}

/* 表单样式 */
.modern-form {
  padding: 10px 0;
}

.modern-form :deep(.el-form-item__label) {
  font-weight: 600;
  color: #34495e;
  font-size: 0.95rem;
}

.modern-date-picker {
  width: 100%;
  border-radius: 12px;
  border: 2px solid #e0e6ed;
  transition: all 0.3s ease;
}

.modern-date-picker:hover {
  border-color: #409eff;
  box-shadow: 0 0 0 4px rgba(64, 158, 255, 0.1);
}

/* 对话框表格 */
.dialog-table {
  border-radius: 12px;
  overflow: hidden;
  border: none;
}

.dialog-table :deep(.el-table__header-wrapper) {
  background: linear-gradient(45deg, #f8f9fa, #e9ecef);
}

/* 加载动画 */
.modern-table :deep(.el-loading-mask) {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(5px);
}

.modern-table :deep(.el-loading-spinner) {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
}

.modern-table :deep(.el-loading-text) {
  color: #409eff;
  font-weight: 600;
  font-size: 1.1rem;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .book-container {
    padding: 15px;
  }

  .page-title {
    font-size: 2rem;
  }

  .form-item {
    margin-right: 10px;
    margin-bottom: 10px;
  }

  .action-group {
    flex-direction: column;
    gap: 5px;
  }

  .action-buttons {
    flex-direction: column;
  }
}

@media (max-width: 480px) {
  .search-form {
    flex-direction: column;
  }

  .form-item {
    margin-right: 0;
    width: 100%;
  }

  .modern-button {
    width: 100%;
    justify-content: center;
  }
}

/* 动画效果 */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.search-card,
.action-card,
.table-card {
  animation: fadeInUp 0.6s ease-out;
}

.search-card {
  animation-delay: 0.1s;
}

.action-card {
  animation-delay: 0.2s;
}

.table-card {
  animation-delay: 0.3s;
}

/* 自定义滚动条 */
.modern-table :deep(.el-table__body-wrapper::-webkit-scrollbar) {
  width: 8px;
  height: 8px;
}

.modern-table :deep(.el-table__body-wrapper::-webkit-scrollbar-track) {
  background: #f1f1f1;
  border-radius: 4px;
}

.modern-table :deep(.el-table__body-wrapper::-webkit-scrollbar-thumb) {
  background: linear-gradient(45deg, #667eea, #764ba2);
  border-radius: 4px;
}

.modern-table :deep(.el-table__body-wrapper::-webkit-scrollbar-thumb:hover) {
  background: linear-gradient(45deg, #764ba2, #667eea);
}
</style>