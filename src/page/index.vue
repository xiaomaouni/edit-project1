<template>
  <section class="decoration-edit">
    <section class="l">
      <ul @dragstart="dragStart" @dragend="dragEnd">
        <li v-for="(val, key, index) in typeList" draggable :data-type="key" :key="index + 1">
          <span :class="val.icon"></span>
          <p>{{val.name}}</p>
        </li>
      </ul>
    </section>
    <section class="c">
      <div class="view-content" @drop="drog" @dragover="dragOver">
        <!-- <Draggable v-model="view" draggable=".item">
          <template v-for="(item, index) in view" :key="index">
            <div :data-index="index" class="item" @click="selectType(index)">
              <template v-if="item.status && item.status == 2">
                <div class="wait"> {{ item.type }} </div>
              </template>
              <template v-else>
                <component :is="typeList[item.type]['com']" :data="item" :className="className[item.tabType]">
                </component>
              </template>
              <i @click="deleteItem($event, index)" class="el-icon-error"></i>
            </div>
          </template>
        </Draggable> -->
      </div>
    </section>

    <section class="r">
      <EditForm :data="props" v-if="isRight"></EditForm>
    </section>

    <div class="submit-btn">
      <el-button @click="submit">提交页面</el-button>
    </div>
  </section>
</template>

<script setup>
  import Draggable from 'vuedraggable'
  // import {
  //   imageUpload
  // } from '@/api/commodity'
  import EditForm from '../components/Edit/index.vue'
  import Product from '../components/View/Product.vue'
  import Images from '../components/View/Images.vue'
  import Banner from '../components/View/Banner.vue'
  import Button from '../components/View/Button.vue'
  import Txt from '../components/View/Txt.vue'
  import {
    reactive,
    ref,
    getCurrentInstance,
    nextTick,
    watch
  } from 'vue';
  let {
    proxy
  } = getCurrentInstance();
  let typeList = reactive({
    banner: {
      name: '轮播图',
      icon: 'el-icon-picture',
      com: Banner
    },
    product: {
      name: '商品',
      icon: 'el-icon-s-goods',
      com: Product
    },
    images: {
      name: '图片',
      icon: 'el-icon-picture',
      com: Images
    },
    txt: {
      name: '文本',
      icon: 'el-icon-notebook-2',
      com: 'Txt'
    },
    button: {
      name: "按钮",
      icon: 'el-icon-turn-off',
      com: 'Button'
    }
  })
  let view = ref([])
  let title = ref('页面标题')
  let type = ref('')
  let index = ref(null)
  let isPush = ref(false)
  let props = ref({})
  let isRight = ref(false)
  let className = reactive({
    1: 'one',
    2: 'two',
    3: 'three',
    4: 'four',
    5: 'five'
  })
  let submit = () => {
    // JSON 转换会丢失 formData
    const form = JSON.parse(JSON.stringify(view.value))
    console.log(form)
    if (form.length == 1) {
      // this.$message.error('请添加模块！')
      console.log('请添加模块！')
      return
    }

    for (let i of form) {
      if (i.data && i.data.length == 0) {
        // this.$message.error('请填写完整信息！')
        console.log('请填写完整信息！')
        return
      }
      if (i.type === 'product') {
        i.data = i.data.map(val => val.productId).join(',')
      }
    }

    // this.$message.success('数据提交成功，请按F12打开控制台查看待提交数据集合！')
    console.log('数据提交成功，请按F12打开控制台查看待提交数据集合！')

    console.log(form) // 提交的数据，根据接口形式修改

    return
  }

  // 切换视图组件
  let selectType = (index) => {
    isRight.value = false
    props.value = view.value[index]
    nextTick(() => isRight.value = true)
  }
  let deleteItem = (e, index) => {
    e.preventDefault()
    e.stopPropagation()
    view.value.splice(index, 1)
    isRight.value = false
    props.value = {}
  }
  // 拖拽类型
  let dragStart = (e) => {
    type.value = e.target.dataset.type
  }
  // 结束拖拽
  let dragEnd = (e) => {
    // this.$delete(view.value[index.value], 'status')
    isPush.value = false
    type.value = null

  }
  // 已放置到指定位置
  let drog = (e) => {
    if (!type.value) { // 内容拖拽
      return
    }
    e.preventDefault()
    e.stopPropagation()
    dragEnd()
  }
  // 移动中
  let dragOver = (e) => {
    if (!type.value) { // 内容拖拽
      return
    }
    e.preventDefault()
    e.stopPropagation()

    let className = e.target.className
    let name = className !== 'view-content' ? 'item' : 'view-content'

    const defaultData = {
      type: type.value, // 组件类型
      status: 2, // 默认状态
      data: [], // 数据
      options: {} // 选项操作
    }

    if (name == 'view-content') {
      if (!isPush.value) {
        index.value = view.value.length
        isPush.value = true
        view.value.push(defaultData)
      }
    } else if (name == 'item') {

      let target = e.target
      let [y, h, curIndex] = [e.offsetY, target.offsetHeight, target.dataset.index]
      let direction = y < (h / 2)

      if (!isPush.value) {
        // Push to Top or Bottom
        if (direction) {
          if (curIndex == 0) {
            view.value.unshift(defaultData)
          } else {
            view.value.splice(curIndex, 0, defaultData)
          }
        } else {
          curIndex = +curIndex + 1
          view.value.splice(curIndex, 0, defaultData)
        }
      } else {
        // Moving
        if (direction) {
          var i = curIndex == 0 ? 0 : curIndex - 1
          var result = view.value[i]['status'] == 2
        } else {
          var i = +curIndex + 1
          var result = view.value.length > i && view.value[i]['status'] == 2
        }

        if (result) return

        const temp = view.value.splice(index.value, 1)
        view.value.splice(curIndex, 0, temp[0])
      }
      index.value = curIndex
      isPush.value = true
    }
  }
</script>
<style lang="less">
  .decoration-edit {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px 0;
    background: #f7f8f9;
    height: calc(100vh - 50px);
    position: relative;

    .l,
    .r {
      width: 340px;
      height: 100%;
      padding: 15px 0;
      background: #fff;
    }

    .l {
      ul {
        margin: 0;
        padding: 0;

        li {
          width: 80px;
          height: 80px;
          display: flex;
          justify-content: center;
          align-items: center;
          flex-direction: column;
          cursor: default;
          list-style: none;
          font-size: 14px;
          color: #666;
          float: left;
          margin: 0 10px;
          border-radius: 6px;
          transition: all .3s;
          cursor: pointer;

          &:hover {
            background: #efefef;
          }

          span {
            display: block;
            font-size: 40px;
            margin-bottom: 8px;
            color: #999;
          }

          p {
            display: block;
            margin: 0;
            font-size: 12px;
          }
        }
      }
    }

    .c {
      width: auto;
      max-width: 1120px;
      position: relative;

      .top-nav {
        position: absolute;
        top: 0;
        background: #fff;
        z-index: 999;
        transition: all .3s;

        & * {
          pointer-events: none;
        }

        &:hover {
          transform: scale(0.95);
          border-radius: 10px;
          overflow: hidden;
          box-shadow: 0 0 10px #afafaf;
        }

        .tit {
          position: absolute;
          left: 50%;
          bottom: 10px;
          transform: translateX(-50%);
        }

        img {
          max-width: 100%;
          image-rendering: -moz-crisp-edges;
          image-rendering: -o-crisp-edges;
          image-rendering: -webkit-optimize-contrast;
          image-rendering: crisp-edges;
          -ms-interpolation-mode: nearest-neighbor;
        }
      }

      .view-content {
        width: 1120px;
        height: 700px;
        background: #f5f5f5;
        overflow-y: auto;
        overflow-x: hidden;
        // padding-top: 72px;
        box-shadow: 0 2px 6px #ccc;

        &::-webkit-scrollbar {
          width: 6px;
        }

        &::-webkit-scrollbar-thumb {
          background: #dbdbdb;
        }

        &::-webkit-scrollbar-track {
          background: #f6f6f6;
        }

        .item {
          transition: all .3s;
          background: #fff;

          &:hover {
            transform: scale(0.95);
            border-radius: 10px;
            box-shadow: 0 0 10px #afafaf;

            .el-icon-error {
              display: block;
            }
          }

          div {
            pointer-events: none;
          }

          .wait {
            background: #deedff;
            height: 35px;
            text-align: center;
            line-height: 35px;
            font-size: 12px;
            color: #666;
          }

          .el-icon-error {
            position: absolute;
            right: -10px;
            top: -6px;
            color: red;
            font-size: 25px;
            cursor: pointer;
            display: none;
            z-index: 9999;
          }
        }
      }
    }

    .submit-btn {
      position: absolute;
      bottom: 30px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 1000000000000;
    }
  }
</style>