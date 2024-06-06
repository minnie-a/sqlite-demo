<template>
  <view>
    <view class="uni-divider uni-divider__content">Demo</view>
    <button @click="openSQL">打开数据库</button>
    <button @click="createTable">创建表</button>
    <button @click="insertTableData">新增表数据</button>
    <button @click="selectTableData">查询表数据</button>
    <button @click="updateTableData">修改表数据</button>
    <button @click="deleteTableData">按条件删除表数据</button>
    <button @click="closeSQL">关闭数据库</button>

    <view class="uni-divider__content" v-for="(item, index) in listData" :key="index">
      <view>名字:{{ item.name }}</view>
      <view>内容:{{ item.content }}</view>
      <view>时间:{{ item.time }}</view>
    </view>
  </view>
</template>

<script>
// 引入封装的 sqlite
import DB from '@/common/sqlite.js'
export default {
  data() {
    return {
      listData: []
    }
  },

  onLoad() {
    this.openSQL()
  },

  methods: {
    // 打开数据库
    openSQL() {
      // 这个是查询有没有打开数据库
      const open = DB.isOpen()
      console.log('数据库状态', open ? '开启' : '关闭')
      if (!open) {
        DB.openSqlite()
          .then((res) => {
            this.showToast('数据库已打开')
          })
          .catch((error) => {
            console.log('数据库开启失败')
          })
      }
    },

    // 关闭数据库
    closeSQL() {
      // 这个是查询有没有打开数据库
      const open = DB.isOpen()
      if (open) {
        DB.closeSqlite()
          .then((res) => {
            this.showToast('数据库已关闭')
          })
          .catch((error) => {
            this.showToast('数据库关闭失败')
          })
      }
    },

    // 创建表
    createTable() {
      const open = DB.isOpen()
      if (open) {
        this.openSQL()
        const sql = '"id" INTEGER PRIMARY KEY AUTOINCREMENT,"name" text,"content" text,"time" text'
        // 创建表 DB.createTable(表名, 表的列)
        DB.createTable('chat', sql)
          .then((res) => {
            this.showToast('创建chat表成功')
          })
          .catch((error) => {
            this.showToast('创建表失败')
          })
      } else {
        this.showToast('数据库未打开')
      }
    },

    // 新增表数据
    insertTableData() {
      const open = DB.isOpen()
      if (open) {
        const arr = [
          {
            name: '小明',
            content: '你好呀'
          },
          {
            name: '小红',
            content: 'HI'
          }
        ]
        arr.map((item) => {
          const time = this.formatDate(new Date().getTime())
          const sql = `'${item.name}','${item.content}','${time}'`
          const condition = "'name','content','time'"
          // 新增 DB.insertTableData(表名, 对应表头列的数据)
          DB.insertTableData('chat', sql, condition)
            .then((res) => {
              this.showToast('新增数据成功')
              this.selectTableData()
            })
            .catch((error) => {
              console.log('失败', error)
            })
        })
      } else {
        this.showToast('数据库未打开')
      }
    },

    // 查询表数据
    selectTableData() {
      const open = DB.isOpen()
      if (open) {
        // 查询表 DB.selectTableData(表名,查询条件列名,查询条件列值)
        DB.selectTableData('chat')
          .then((res) => {
            console.log('contact表数据', res)
            this.listData = res
          })
          .catch((error) => {
            console.log('查询失败', error)
          })
      } else {
        this.showToast('数据库未打开')
      }
    },

    // 修改表数据
    updateTableData() {
      const open = DB.isOpen()
      if (open) {
        const time = this.formatDate(new Date().getTime())
        const data = `content = '我被修改了',time = '${time}'`
        // 修改表数据 DB.updateTableData(表名, 要修改的列名=修改后列值, 修改条件的列名, 修改条件的列值)
        DB.updateTableData('chat', data, 'name', '小明')
          .then((res) => {
            this.showToast('更新chat表成功')
            this.selectTableData()
          })
          .catch((error) => {
            console.log('修改失败', error)
          })
      } else {
        this.showToast('数据库未打开')
      }
    },

    // 删除表数据
    deleteTableData() {
      const open = DB.isOpen()
      if (open) {
        // 删除表 DB.deleteTableData(表名,查询条件列名,查询条件列值)
        DB.deleteTableData('chat', 'name', '小红')
          .then((res) => {
            this.showToast('删除表数据成功')
            this.selectTableData()
          })
          .catch((error) => {
            console.log('删除失败', error)
          })
      } else {
        this.showToast('数据库未打开')
      }
    },

    // 提示框
    showToast: function (str) {
      uni.showToast({
        icon: 'none',
        title: str,
        mask: true
      })
    },

    // 时间戳转年月日
    formatDate(data) {
      const now = new Date(data)
      const year = now.getFullYear() //取得4位数的年份
      const month = now.getMonth() + 1 < 10 ? '0' + (now.getMonth() + 1) : now.getMonth() + 1
      const date = now.getDate() < 10 ? '0' + now.getDate() : now.getDate()
      const hour = now.getHours() < 10 ? '0' + now.getHours() : now.getHours()
      const minute = now.getMinutes() < 10 ? '0' + now.getMinutes() : now.getMinutes()
      const second = now.getSeconds() < 10 ? '0' + now.getSeconds() : now.getSeconds()
      return year + '-' + month + '-' + date + ' ' + hour + ':' + minute + ':' + second
    }
  }
}
</script>
