<template>
  <div class="hello">
    <h1>indexedDB快速入门</h1>
    <ul>
      <li>1.创建及打开数据库，建立表和索引</li>
      <li>2.新增数据</li>
      <li>3.查询数据: 主键查询、游标查询、索引查询、游标和索引查询</li>
      <li>4.更新数据</li>
      <li>5.删除数据：主键删除、游标和索引删除</li>
      <li>6.关闭和删除数据库</li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      indexedDB: {},
      objectDB: {},
      dataList: [],
      dataItem: {}
    }
  },
  async mounted () {
    this.indexedDB = window.indexedDB || window.webkitindexedDB // 兼容indexedDB
    if (!this.indexedDB) {
      console.log('你的浏览器不支持IndexedDB')
    }
    // 1. 打开数据库 数据库存在 ？onsuccess/onerror ：onupgradeneeded => onsuccess/onerror
    let db = await this.openDB()

    // 2.新增数据
    // this.getDataList()
    // this.dataList.forEach(ele => {
    //   this.addData(db, 'person', ele)
    // })

    // 3. 查询数据  主键查询 游标查询 索引查询 游标和索引查询
    // （1）主键查询
    this.dataItem = await this.getDataByKey(db, 'person', 9)
    // (2)游标查询
    // this.cursorGetData(db, 'person')
    // (3) 索引查询
    // this.getDataByIndex(db, 'person', 'address', '云南省')
    // （4）游标和索引查询
    // this.cursorGetDataByIndex(db, 'person', 'address', '云南省')

    // 4.更新数据
    // this.dataItem.flag = 9
    // this.updateData(db, 'person', this.dataItem)

    // 5.删除数据
    // (1) 主键
    // this.deleteData(db, 'person', 8)
    // （2）游标删除
    // this.cursorDeleteData(db, 'person', 'address', '云南省')

    // 6.关闭和删除数据库
    this.closeDB(db)
    this.deletedb()
  },
  methods: {
    // 7.更新数据 db：数据库实例 tableName：表名称 data：数据
    updateData: function (db, storename, data) {
      var request = db.transaction([storename], 'readwrite') // 事务对象
        .objectStore(storename) // 仓库对象
        .put(data)

      request.onsuccess = function () {
        console.log('数据更新成功')
      }

      request.onerror = function () {
        console.log('数据更新失败')
      }
    },
    // 8.删除数据 db：数据库实例 tableName：表名称 id：主键值
    deleteData: function (db, storename, id) {
      var request = db.transaction([storename], 'readwrite') // 事务
        .objectStore(storename) // 仓库对象
        .delete(id)

      request.onsuccess = function () {
        console.log('数据删除成功')
      }

      request.onerror = function () {
        console.log('数据删除失败')
      }
    },
    // 9.通过索引和游标删除指定的数据 db:数据库实例  storename:表名称   indexName：索引名  indexValue：索引值
    cursorDeleteData: function (db, storename, indexName, indexValue) {
      var store = db.transaction(storename, 'readwrite').objectStore(storename) // 仓库对象
      var request = store.index(indexName) // 索引对象
        .openCursor(IDBKeyRange.only(indexValue)) // 指针对象
      request.onsuccess = function (e) {
        var cursor = e.target.result
        var deleteRequest
        if (cursor) {
          deleteRequest = cursor.delete()// 请求删除当前项
          deleteRequest.onerror = function () {
            console.log('游标删除该记录失败')
          }
          deleteRequest.onsuccess = function () {
            console.log('游标删除该记录成功')
          }
          cursor.continue()
        }
      }
      request.onerror = function (e) {
      }
    },
    // 10.关闭数据库 db：数据库实例
    closeDB: function (db) {
      db.close()
      console.log('数据库已关闭')
    },
    // 11.删除数据库 dbname：数据库名称
    deletedb: function () {
      var self = this
      let deleteRequest = self.indexedDB.deleteDatabase('databaseName')
      deleteRequest.onerror = function (event) {
        console.log('删除失败')
      }
      deleteRequest.onsuccess = function (event) {
        console.log('删除成功')
      }
    },
    // 1.打开数据库
    openDB () {
      return new Promise((resolve, reject) => {
        let db
        const request = this.indexedDB.open('databaseName', 1) // 事务对象  databaseName:数据库的名字 version:数据库的版本
        request.onsuccess = function (event) {
          db = event.target.result // 数据库对象
          console.log('数据库打开成功')
          resolve(db)
        }

        request.onerror = function (event) {
          console.log('数据库打开报错')
        }

        request.onupgradeneeded = function (event) { // 数据库创建或升级的时候会触发
          console.log('onupgradeneeded')
          db = event.target.result // 数据库对象
          var objectStore
          if (!db.objectStoreNames.contains('person')) {
            objectStore = db.createObjectStore('person', { keyPath: 'id' }) // 创建表
            // objectStore = db.createObjectStore('person', { autoIncrement: true }) // 创建表
            objectStore.createIndex('name', 'name', { unique: false }) // 创建索引 可以让你搜索任意字段
            objectStore.createIndex('address', 'address', { unique: false })
            objectStore.createIndex('nameAddr', ['name', 'address'], { unique: false })
            objectStore.createIndex('flag', 'flag', { unique: false })
          }
        }
      })
    },
    // mock获取数据
    getDataList () {
      // 使用 Mock
      var Mock = require('mockjs')
      let mockList = Mock.mock({
        'list|20': [{
          'id|+1': 1, // 属性 id 是一个自增数，起始值为 1，每次增 1
          'name|1': '@cname', // 生成姓名
          'account|1000-9999': 1, // 生成姓名
          'email': '@email', // 生成邮箱
          'address': '@province', // 生成地址 省市县
          'flag|1': 1
        }]
      })
      this.dataList = mockList.list
      console.log('this.dataList', this.dataList)
    },
    // 2.新增数据 db：数据库实例 storename：仓库名称 data：数据
    addData (db, storename, data) {
      var request = db.transaction([storename], 'readwrite') // 事务对象 指定表格名称和操作模式（"只读"或"读写"）
        .objectStore(storename) // 仓库对象
        .add(data)

      request.onsuccess = function (event) {
        console.log('数据写入成功')
      }

      request.onerror = function (event) {
        console.log('数据写入失败')
        throw new Error(event.target.error)
      }
    },
    // 查询数据
    // 3.通过主键读取数据 storename：表名称 key 主键值
    getDataByKey: function (db, storename, key) {
      return new Promise((resolve, reject) => {
        var transaction = db.transaction([storename]) // 事务
        var objectStore = transaction.objectStore(storename) // 仓库对象
        var request = objectStore.get(key)

        request.onerror = function (event) {
          console.log('事务失败')
        }

        request.onsuccess = function (event) {
          console.log('主键查询结果: ', request.result)
          resolve(request.result)
        }
      })
    },
    // 4.通过游标查询记录 db 数据库实例  storename 仓库/表名称
    cursorGetData: function (db, storename) {
      let list = []
      var store = db.transaction(storename, 'readwrite') // 事务
        .objectStore(storename) // 仓库对象
      var request = store.openCursor() // 指针对象
      request.onsuccess = function (e) {
        var cursor = e.target.result
        if (cursor) { // 必须要检查
          list.push(cursor.value)
          cursor.continue()// 遍历了存储对象中的所有内容
        } else {
          console.log('游标查询结果：', list)
        }
      }
    },
    // 5.通过索引读取数据 db 数据库实例 storename：表名称 indexName：索引名称 indexValue：索引值
    getDataByIndex: function (db, storename, indexName, indexValue) {
      var store = db.transaction(storename, 'readwrite') // 事务
        .objectStore(storename) // 仓库对象
      var request = store.index(indexName) // 索引对象
        .get(indexValue)
      request.onerror = function () {
        console.log('事务失败')
      }
      request.onsuccess = function (e) {
        var result = e.target.result
        console.log('索引查询结果：', result)
      }
    },
    // 6.通过索引游标查询记录  db:数据库实例 storename 仓库/表名称  indexName：索引名  indexValue：索引值
    cursorGetDataByIndex: function (db, storename, indexName, indexValue) {
      let list = []
      var store = db.transaction(storename, 'readwrite').objectStore(storename) // 仓库对象
      var request = store.index(indexName) // 索引对象
        .openCursor(IDBKeyRange.only(indexValue)) // 指针对象
      request.onsuccess = function (e) {
        var cursor = e.target.result
        if (cursor) { // 必须要检查
          list.push(cursor.value)
          cursor.continue()// 遍历了存储对象中的所有内容
        } else {
          console.log('游标索引查询结果：', list)
        }
      }
      request.onerror = function (e) {
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
  text-align: left
}
li {
  margin: 15px;
  border: 1px solid #eee;
  padding: 10px;
  background: #eee;
}
a {
  color: #42b983;
}
</style>
