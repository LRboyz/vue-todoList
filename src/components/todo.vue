<template>
  <div>
    <el-container style="height: 100vh; border: 1px solid #eee;">
      <el-aside width="200px" style="background-color: rgb(238, 241, 246);">
        <el-menu :default-openeds="['1', '3']">
          <el-submenu index="1">
            <template slot="title">
              ToDoList
            </template>
            <el-menu-item-group>
              <el-menu-item index="1-1">页面1</el-menu-item>
              <el-menu-item index="1-2">页面2</el-menu-item>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-aside>

      <el-container>
        <el-header style="text-align: right; font-size: 12px;">
          <el-dropdown>
            <i class="el-icon-setting" style="margin-right: 15px;"></i>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>查看</el-dropdown-item>
              <el-dropdown-item>新增</el-dropdown-item>
              <el-dropdown-item>删除</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <span>王小虎</span>
        </el-header>

        <el-card>
          <!-- 右侧头部 -->
          <div class="table">
            <h2>Todo 待办事项</h2>
          </div>
          <!-- table头部 -->
          <div>
            <el-form
              ref="todo"
              :model="todo"
              :rules="rules"
              label-width="80px"
              class="form"
              @submit.native.prevent="add(todo)"
            >
              <el-form-item prop="content">
                <el-input
                  v-model="todo.content"
                  placeholder="添加任务至Todo"
                  class="input-with-select"
                  name="content"
                >
                  <el-select
                    slot="prepend"
                    v-model="todo.category"
                    placeholder="请选择"
                    style="width: 110px;"
                  >
                    <el-option
                      v-for="(item, index) in types"
                      :value="item"
                      :key="index"
                    >
                      {{ item }}
                    </el-option>
                  </el-select>
                  <el-button
                    slot="append"
                    native-type="submit"
                    icon="el-icon-circle-plus-outline"
                  />
                </el-input>
              </el-form-item>
            </el-form>
          </div>
          <!-- table主体 -->
          <el-main>
            <el-table :data="tableData" border>
              <el-table-column
                prop="number"
                label="序号"
                width="80"
                type="index"
                height="100"
              />
              <el-table-column prop="category" label="类别" width="150" />
              <el-table-column prop="content" label="内容" width="300" />
              <el-table-column prop="status" label="状态">
                <template slot-scope="scope">
                  <i
                    :class="
                      scope.row.status == true
                        ? 'green el-icon-circle-check'
                        : 'red el-icon-circle-close'
                    "
                  ></i>
                  <!-- <div
                    :class="{ green: scope.row.status }"
                    class="state"
                    @click="ChangeState(scope.row.id, scope.row.status)"
                  /> -->
                  <span :class="scope.row.status == true ? 'green' : 'red'">
                    {{ scope.row.status == true ? '已完成' : '未完成' }}
                  </span>
                </template>
              </el-table-column>
              <el-table-column prop="time" label="更新时间" sortable />
              <el-table-column prop="id" label="操作" width="220">
                <template slot-scope="scope">
                  <el-button
                    type="primary"
                    icon="el-icon-edit"
                    size="mini"
                    @click="Edit(scope.row)"
                  >
                    编辑
                  </el-button>

                  <el-button
                    type="danger"
                    icon="el-icon-delete"
                    size="mini"
                    @click="Delete(scope.row.id)"
                  >
                    删除
                  </el-button>
                  <el-dialog :visible.sync="dialogFormVisible" title="修改数据">
                    <el-form :rules="rules" label-position="left">
                      <el-form-item
                        :label-width="formLabelWidth"
                        label="Todo内容:"
                      >
                        <el-input
                          v-model="EditTodo.content"
                          autocomplete="off"
                          required
                        />
                      </el-form-item>
                      <el-form-item
                        :label-width="formLabelWidth"
                        label="请选择分类:"
                      >
                        <el-select
                          v-model="EditTodo.category"
                          placeholder="请选择类型"
                        >
                          <el-option
                            v-for="(item, index) in types"
                            :value="item"
                            :key="index"
                          >
                            {{ item }}
                          </el-option>
                        </el-select>
                      </el-form-item>
                      <el-form-item
                        :label-width="formLabelWidth"
                        label="请选择状态:"
                      >
                        <el-select
                          v-model="EditTodo.status"
                          placeholder="请选择状态"
                        >
                          <el-option
                            v-for="(item, index) in [
                              { value: true, label: '完成' },
                              { value: false, label: '未完成' },
                            ]"
                            :value="item.value"
                            :label="item.label"
                            :key="index"
                          >
                            <!-- {{ item }} -->
                          </el-option>
                        </el-select>
                      </el-form-item>
                    </el-form>
                    <div slot="footer">
                      <el-button @click="dialogFormVisible = false">
                        取 消
                      </el-button>
                      <el-button
                        type="primary"
                        @click="ChangeEdit(EditTodo.id)"
                      >
                        确 定
                      </el-button>
                    </div>
                  </el-dialog>
                </template>
              </el-table-column>
            </el-table>
          </el-main>
        </el-card>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import { getJSONStorageReader } from '../utils/localstorage'

const localTodoList = getJSONStorageReader('todolist')

export default {
  name: 'TodoList',
  data() {
    return {
      todo: {
        number: '',
        category: '',
        content: '',
        time: new Date().toLocaleString(),
      },
      EditTodo: {},
      types: ['全部', '工作', '日常', '娱乐'],
      rules: {
        content: [
          { required: true, message: '请输入内容在提交哦', trigger: 'blur' },
          { min: 2, max: 20, message: '长度在2-20位之间', trigger: 'blur' },
        ],
        category: [{ required: true, message: '请选择类型哦' }],
      },
      dialogTableVisible: false,
      dialogFormVisible: false,
      formLabelWidth: '120px',
      tableData: [],
    }
  },
  created() {
    this.fetch()
  },
  methods: {
    // 生成唯一ID
    guid() {
      return Number(
        Math.random().toString().substr(3, 3) + Date.now(),
      ).toString(36)
    },

    fetch() {
      const todolist = localTodoList.get() || []
      this.tableData = todolist
    },
    // 添加数据
    add() {
      this.$refs['todo'].validate((valid) => {
        if (valid) {
          this.tableData.unshift({
            id: this.guid(),
            category: this.todo.category,
            content: this.todo.content,
            status: false,
            time: new Date().toLocaleString(),
          })
          localTodoList.set(this.tableData)
        } else {
          this.$notify.error({
            title: '错误',
            message: '校验失败，请重新输入！',
          })
          return false
        }
      })
    },
    // 进入编辑
    Edit(row) {
      //row
      // console.log(row)
      this.dialogFormVisible = true
      this.EditTodo = {
        id: row.id,
        content: row.content,
        category: row.category,
        status: row.status,
      }
      // console.log(this.tableData)
    },
    // 修改数据
    ChangeEdit(id) {
      // console.log(id, this.EditTodo)
      this.tableData.map((item) => {
        if (item.id == id) {
          item.id = this.EditTodo.id
          item.content = this.EditTodo.content
          item.category = this.EditTodo.category
          item.status = this.EditTodo.status
          item.time = new Date().toLocaleString()
        }
      })
      localTodoList.set(this.tableData)
      this.dialogFormVisible = false
    },
    // 删除数据
    Delete(id) {
      console.log(id)

      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          this.tableData.map((item, index) => {
            if (item.id == id) {
              this.tableData.splice(index, 1)
              localTodoList.set(this.tableData)
            }
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          })
        })
    },
  },
}
</script>

<style>
.el-header {
  background-color: #b3c0d1;
  color: #333;
  line-height: 60px;
}

.el-aside {
  color: #333;
}
.table {
  text-align: center;
  line-height: 30px;
  color: rgb(4, 134, 240);
  margin-bottom: 20px;
}
.table .look {
  width: 100%;
  list-style: none;
  display: flex;
  justify-content: flex-start;
}
.table .look li {
  margin-right: 10px;
}
li:hover {
  color: #2276f3;
  cursor: pointer;
}
.state {
  display: inline-block;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: red;
  transition: all 0.5s;
  border: 2px solid #c5c5c5;
}
.blue {
  color: rgb(4, 134, 240);
}
.green {
  transition: all 0.5s;
  color: rgb(98, 215, 44);
}
.red {
  transition: all 0.5s;
  color: red;
}
</style>
