<template>
  <div class="hello">
    <h1>indexedDB快速入门</h1>
    <ul>
      <li>1.创建及打开数据库，建立表和索引</li>
      <li>2.新增数据</li>
      <li>3.查询数据，根据游标和索引</li>
      <li>4.更新数据，key</li>
      <li>5.删除数据，key或索引</li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      indexedDB: {},
      objectDB: {}
    }
  },
  mounted () {
    this.indexedDB = window.indexedDB || window.webkitindexedDB // 兼容indexedDB
    this.openDB()
  },
  methods: {
    openDB () { // 1. 数据库是否存在，是 => 回调函数； 否 => 触发onupgradeneeded函数，回调函数
      const request = window.indexedDB.open('databaseName', 1) // databaseName:数据库的名字 version:数据库的版本
      request.onerror = function (event) {
        console.log('数据库打开报错')
      }

      request.onsuccess = function (event) {
        this.objectDB = request.result
        console.log('数据库打开成功')
      }

      request.onupgradeneeded = function (event) { // 数据库创建或升级的时候会触发
        console.log('onupgradeneeded触发')
        this.objectDB = event.target.result
        var objectStore
        if (!this.objectDB.objectStoreNames.contains('person')) { // 创建数据表
          objectStore = this.objectDB.createObjectStore('person', { keyPath: 'id' })
          objectStore.createIndex('name', 'name', { unique: false })
          objectStore.createIndex('email', 'email', { unique: true })
        }
      }
    },
    addData () {
      var request = this.objectDB.transaction(['person'], 'readwrite')
        .objectStore('person')
        .add({ id: 1, name: '张三', age: 24, email: 'zhangsan@example.com' })

      request.onsuccess = function (event) {
        console.log('数据写入成功')
      }

      request.onerror = function (event) {
        console.log('数据写入失败')
        throw new Error(event.target.error)
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
  margin: 10px;
}
a {
  color: #42b983;
}
</style>
