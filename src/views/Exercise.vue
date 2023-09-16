<template>
  <div class="exercise">
    <div class="header">
      {{ getTitle() }}
    </div>
    <div class="obj" v-if="$route.query.type == 3 && errArr.length === 0">
      <el-tag type="success">暂无错题</el-tag>
    </div>
    <div v-else>
      <div class="obj">
        <h2>{{ index + 1 }}.{{ row.title }}</h2>
        <div v-if="row.type == 0">
          <div class="itemWarp" v-for="(item, index) in row.items" :key="item.value">
            <el-radio v-model="radio" :label="item.value">{{ orderArr[index] }}.{{ item.label }}</el-radio>
          </div>
        </div>
        <div class="btnWarp" v-else>
          <el-checkbox-group v-model="checkList">
            <el-checkbox class="checkbox" v-for="(item, index) in row.items" :key="item.value" :label="item.value">{{
              orderArr[index] }}.{{ item.label }}</el-checkbox>
          </el-checkbox-group>
        </div>
      </div>
      <div class="btnWarp">
        <el-button type="primary" @click="check">确定</el-button>
      </div>
      <div class="btnWarp" v-if="res != null">
        <el-tag type="success" v-if="res">正确</el-tag>
        <el-tag type="danger" v-else>错误，正确答案：{{ answer }}</el-tag>
      </div>
      <div class="btnWarp">
        <el-button @click="prev">上一题</el-button>
        <el-button @click="next">下一题</el-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      orderArr: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'],
      radio: '',
      checkList: [],
      index: 0,
      row: {},
      res: null,
      answer: '',
      errArr: [],
      list: [
        {
          title: '对未取得驾驶证驾驶机动车的，追究其法律责任',
          // type 0 单选 1多选
          type: 0,
          answer: '1',
          items: [
            {
              value: '1',
              label: '正确'
            },
            {
              value: '2',
              label: '错误'
            }
          ]
        },
        {
          title: '驾驶机动车应当随身携带哪种证件？',
          // type 0 单选 1多选
          type: 0,
          answer: '2',
          items: [
            {
              value: '1',
              label: '身份证'
            },
            {
              value: '2',
              label: '驾驶证'
            },
            {
              value: '3',
              label: '职业资格'
            },
            {
              value: '4',
              label: '工作证'
            }
          ]
        },
        {
          title: '新形势下加强和规范党内政治生活要求，要在全党努力形成生动活泼的政治局面。（    ）',
          // type 0 单选 1多选
          type: 1,
          answer: '124',
          items: [
            {
              value: '1',
              label: '又有集中又有民主'
            },
            {
              value: '2',
              label: '又有纪律又有自由'
            },
            {
              value: '3',
              label: '又有组织又有个人'
            },
            {
              value: '4',
              label: '又有统一意志又有个人心情舒畅'
            }
          ]
        },
      ]
    }
  },
  created() {
    const { type } = this.$route.query
    switch (type) {
      // 顺序练习
      case '1':
        if (this.getItem('orderIndex') == null) {
          this.index = 0
        } else {
          this.index = Number(this.getItem('orderIndex'))
        }
        break
      // 随机练习
      case '2':
        this.randomEvent()
        break
      // 错题库练习
      case '3':
        const errArrStr = this.getItem('errArr')
        if (errArrStr) {
          const errArr = JSON.parse(errArrStr)
          this.errArr = errArr
          if(this.getItem('errIndex') == null) {
            this.index = Number(errArr[0])
          } else {
            this.index = Number(this.getItem('errIndex'))
          }
        }
        break

      default:
        break
    }
    // 获取index
    // this.index = 0
    this.row = this.list[this.index]
  },
  methods: {
    check() {
      if (this.row.type == 0) {
        // 单选
        if (!this.radio) {
          this.$message('请选择')
          return
        }
        if (this.radio == this.row.answer) {
          this.res = true
        } else {
          this.res = false
          this.answer = this.orderArr[this.row.answer - 1]
          // 加入错题库
          this.pushErrors()
        }
      } else {
        // 多选
        if (this.checkList.length == 0) {
          this.$message('请选择')
          return
        }
        const checkStr = (this.checkList.sort((a, b) => a - b)).join('')
        if (checkStr == this.row.answer) {
          this.res = true
        } else {
          this.res = false
          const answerArr = this.row.answer.split('')
          answerArr.forEach(item => {
            this.answer += this.orderArr[item - 1]
          })
          // 加入错题库
          this.pushErrors()
        }
      }
    },
    // 加入错题库
    pushErrors() {
      const errArr = this.getItem('errArr')
      if (errArr) {
        let setArr = JSON.parse(errArr)
        if (!setArr.includes(this.index)) {
          setArr.push(this.index)
          this.setItem('errArr', setArr.sort((a, b) => a - b))
        }
      } else {
        const setArr = [this.index]
        this.setItem('errArr', setArr)
      }
    },
    prev() {
      const { type } = this.$route.query
      switch (type) {
        case '1':
            if (this.index === 0) {
              return
            }
            this.index--
            this.setItem('orderIndex', this.index)
          break
        case '2':
          this.randomEvent()
          break
        case '3':
          let eIndex = this.errArr.map(item => item).indexOf(this.index)
          if(eIndex == 0) {
            return
          }
          this.index = this.errArr[eIndex - 1]
          this.setItem('errIndex', this.index)
          break
        default:
          break
      }
      this.initData()
      this.row = this.list[this.index]
    },
    next() {
      const { type } = this.$route.query
      switch (type) {
        case '1':
          if (this.list.length == this.index + 1) {
            return
          }
          this.index++
          this.setItem('orderIndex', this.index)
          break
        case '2':
          this.randomEvent()
          break
        case '3':
          let eIndex = this.errArr.map(item => item).indexOf(this.index)
          if(this.errArr.length == eIndex + 1) {
            return
          }
          this.index = this.errArr[eIndex + 1]
          this.setItem('errIndex', this.index)
          break
        default:
          break
      }
      this.initData()
      this.row = this.list[this.index]
    },
    initData() {
      this.radio = ''
      this.checkList = []
      this.res = null
      this.answer = ''
    },
    getItem(str) {
      return localStorage.getItem(str)
    },
    setItem(name, val) {
      if (typeof val == 'object') {
        localStorage.setItem(name, JSON.stringify(val))
      } else {
        localStorage.setItem(name, val)
      }
    },
    getTitle() {
      const { type } = this.$route.query
      if(type == '1') {
        return '顺序练习'
      }
      if(type == '2') {
        return '随机练习'
      }
      if(type == '3') {
        return '错题库'
      }

    },
    randomEvent() {
      const len = this.list.length
      const anum = (len - 0) * Math.random() + 0
      this.index = Math.floor(anum)
    }
  }
}
</script>
<style lang="less" scoped>
.exercise {
  padding: 40px 20px;
  padding-top: 0;
}

.itemWarp {
  margin-top: 16px;
}

.btnWarp {
  margin-top: 20px;
}

.checkbox {
  display: block;
  margin-top: 16px;
}
.header {
  height: 50px;
  line-height: 50px;
  color: #fff;
  background: #409EFF;
  text-align: center;
  margin: 20px 0;
}
</style>