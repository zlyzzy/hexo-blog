---
title: element-ui tree 节点过滤加载对应子节点方法
keywords: element-ui,tree,vue
description: element-ui tree 节点过滤加载对应子节点方法
tags: Vue
categories: Vue
---

element-ui tree 节点过滤加载对应子节点方法,官网例子，不会返回过滤节点
的子节点，这也是总结这篇博客的原因。


```
//官网例子
<el-input
  placeholder="输入关键字进行过滤"
  v-model="filterText">
</el-input>

<el-tree
  class="filter-tree"
  :data="data2"
  :props="defaultProps"
  default-expand-all
  :filter-node-method="filterNode"
  ref="tree2">
</el-tree>

<script>
  export default {
    watch: {
      filterText(val) {
        this.$refs.tree2.filter(val);
      }
    },

    methods: {
    //不会返回匹配的node的子节点
      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1;
      }
    },

    data() {
      return {
        filterText: '',
        data2: [{
          id: 1,
          label: '一级 1',
          children: [{
            id: 4,
            label: '二级 1-1',
            children: [{
              id: 9,
              label: '三级 1-1-1'
            }, {
              id: 10,
              label: '三级 1-1-2'
            }]
          }]
        }, {
          id: 2,
          label: '一级 2',
          children: [{
            id: 5,
            label: '二级 2-1'
          }, {
            id: 6,
            label: '二级 2-2'
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1'
          }, {
            id: 8,
            label: '二级 3-2'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label'
        }
      };
    }
  };
</script>

```

+ 此时可以实现，当点击搜索时，只会搜索到当前节点包含该搜索字段filterText的树渲染
+ 而我们一般实际业务中，需要搜索到其下所有的子节点
+ 实现方法如下（修改filterNode方法即可，注意注意：filterNode方法有三个参数）

```
filterNode(value,data,node) {
  //如果有三级
  if (!value) return true;
  let if_one = node.data.label.indexOf(value) !== -1
  let if_two = node.parent && node.parent.data && node.parent.data.label && (node.parent.data.label.indexOf(value) !== -1)
  let if_three = node.parent && node.parent.parent && node.parent.parent.data && node.parent.parent.data.label && (node.parent.parent.data.label.indexOf(value) !== -1)
  let result_one = false
  let result_two = false
  let result_three = false
  if(node.level === 1) {
    result_one = if_one
  }else if(node.level === 2) {
    result_two = if_one || if_two
  }else if(node.level === 3) {
    result_three = if_one || if_two || if_three
  }
  return result_one || result_two || result_three
}

//优化之后的代码 不管有几级都可以适用
filterNode(value,data,node) {
  if(!value){
    return true;
  }
  let level = node.level;
  let _array = [];//这里使用数组存储 只是为了存储值。
  this.getReturnNode(node,_array,value);
  let result = false;
  _array.forEach((item)=>{
    result = result || item;
  });
  return result;
},
getReturnNode(node,_array,value){
 let isPass = node.data &&  node.data.ruleName && node.data.ruleName.indexOf(value) !== -1;
 isPass?_array.push(isPass):'';
 this.index++;
 console.log(this.index)
 if(!isPass && node.level!=1 && node.parent){
  this.getReturnNode(node.parent,_array,value);
 }
}


//如果不明白上面的写法 可以看下面详解的写法 


 // 触发页面显示配置的筛选
      filterNode(value, data, node) {
        // 如果什么都没填就直接返回
        if (!value) return true;
        // 如果传入的data和node.data中的ruleCode相同说明是匹配到了
        if (data.ruleName.indexOf(value) !== -1) {
          return true;
        }
        // 否则要去判断它是不是选中节点的子节点
        return this.checkBelongToChooseNode(value, data, node);
      },
      // 判断传入的节点是不是选中节点的子节点
      checkBelongToChooseNode(value, data, node) {
        const level = node.level;
        // 如果传入的节点本身就是一级节点就不用校验了
        if (level === 1) {
          return false;
        }
        // 先取当前节点的父节点
        let parentData = node.parent;
        // 遍历当前节点的父节点
        let index = 0;
        while (index < level - 1) {
          // 如果匹配到直接返回
          if (parentData.data.ruleName.indexOf(value) !== -1) {
            return true;
          }
          // 否则的话再往上一层做匹配
          parentData = parentData.parent;
          index ++;
        }
        // 没匹配到返回false
        return false;
      },


``` 