<template>
  <div class="treeDisplay">
    <span style="grid-area: title" class="title">表达式二叉树</span>
    <div style="grid-area: treeDisplay" v-html="this.resultGraph" class="container"></div>
    <span style="grid-area: text-pre" class="text">前缀表达式:</span>
    <span style="grid-area: text-in" class="text">中缀表达式:</span>
    <span style="grid-area: text-post" class="text">后缀表达式:</span>
    <span style="grid-area: preOrder" class="expression">{{ preOrderExpression.join(" ") }}</span>
    <span style="grid-area: inOrder" class="expression">{{ inOrderExpression.join(" ") }}</span>
    <span style="grid-area: postOrder" class="expression">{{ postOrderExpression.join(" ") }}</span>
  </div>
</template>

<script>

export default {
  name: "TheTreeDisplay",
  props: {
    expressionTree: {
      type: Object
    }
  },
  data() {
    return {
      resultGraph: '',
      preOrderExpression: [],
      inOrderExpression: [],
      postOrderExpression: []
    }
  },
  watch: {
    expressionTree(newExp) {
      console.log("Length: " + Object.keys(newExp).length);
      if (Object.keys(newExp).length === 0) {
        this.resultGraph = '';
        this.preOrderExpression = [];
        this.inOrderExpression = [];
        this.postOrderExpression = [];
      }
      else {
        this.treeTraverse(newExp);
        this.resultGraph = this.treeToHtml(newExp);
      }
    }
  },
  methods: {
    treeTraverse(root) {
      console.log("前序遍历：");
      this.preOrderExpression = [];
      this.preOrderTraverse(root);
      console.log(this.preOrderExpression);
      console.log("中序遍历：");
      this.inOrderExpression = [];
      this.inOrderTraverse(root);
      console.log(this.inOrderExpression);
      console.log("后序遍历：");
      this.postOrderExpression = [];
      this.postOrderTraverse(root);
      console.log(this.postOrderExpression);
      this.$emit("postOrderExpressionChanged", this.postOrderExpression);
    },
    preOrderTraverse(root) {
      const OP_REG = /\d|\./ // 操作数 正则表达式
      if (OP_REG.test(root)) this.preOrderExpression.push(root);
      else {
        this.preOrderExpression.push(root.opr);
        this.preOrderTraverse(root.op1);
        this.preOrderTraverse(root.op2);
      }
    },
    inOrderTraverse(root) {
      const OP_REG = /\d|\./ // 操作数 正则表达式
      if (OP_REG.test(root)) this.inOrderExpression.push(root);
      else {
        this.inOrderTraverse(root.op1);
        this.inOrderExpression.push(root.opr);
        this.inOrderTraverse(root.op2);
      }
    },
    postOrderTraverse(root) {
      const OP_REG = /\d|\./ // 操作数 正则表达式
      if (OP_REG.test(root)) this.postOrderExpression.push(root);
      else {
        this.postOrderTraverse(root.op1);
        this.postOrderTraverse(root.op2);
        this.postOrderExpression.push(root.opr);
      }
    },
    treeToHtml(root) {
      if (root) {
        const OP_REG = /\d|\./ // 操作数 正则表达式
        if (OP_REG.test(root)) {
          console.log("Leaf: " + root);
          return `
        <div class="tree">
          <span><span>${root}</span></span>
        </div>
      `
        } else {
          console.log("Operator: " + root.opr);
          return `
        <div class="tree">
          <span><span>${root.opr}</span></span>
          ${this.treeToHtml(root.op1)}
          ${this.treeToHtml(root.op2)}
        </div>
      `
        }
      } else {
        return ''
      }
    }
  }
}

</script>

<style>

.treeDisplay {
  width: calc(80vw - 450px);
  position: absolute;
  float: left;
  top: 5vh;
  bottom: 10vh;
  right: 5vw;

  display: grid;
  grid-template-columns: 200px auto;
  grid-template-rows: 40px auto repeat(3, 40px);
  grid-template-areas:
  "title title"
  "treeDisplay treeDisplay"
  "text-pre preOrder"
  "text-in inOrder"
  "text-post postOrder";

  border-radius: 11px;
  background: linear-gradient(145deg, #ffffff, #e6e6e6);
  box-shadow: 11px 11px 30px #c9c9c9, -11px -11px 30px #ffffff;
}

.treeDisplay .title {
  text-align: center;
  border-bottom: 1px solid #BDBDBD;
  line-height: 40px;
  font-size: 30px;
  font-family: Helvetica, serif;
  padding: 0 20px;
  overflow-x: auto;
}

.treeDisplay .container {
  padding: 10px 20px;
  white-space: nowrap;
  overflow: auto;
}

.treeDisplay .text {
  text-align: left;
  border-top: 1px solid #BDBDBD;
  border-radius: 1px;
  background: #fafafa;
  line-height: 40px;
  font-size: 30px;
  font-family: Helvetica, serif;
  padding-left: 20px;
  white-space: nowrap;
  overflow-x: auto;
}

.treeDisplay .expression {
  text-align: left;
  border-top: 1px solid #BDBDBD;
  border-radius: 1px;
  background: #fafafa;
  line-height: 40px;
  font-size: 30px;
  font-family: Helvetica, serif;
  padding-right: 20px;
  white-space: nowrap;
  overflow-x: auto;
}

div.tree {
  display: flex;
  flex-wrap: wrap; /*span需要独占一行，所以此flex布局必须要折行显示*/
  align-items: flex-start;
}

.tree > .tree {
  width: 50%;
}

div.tree > span {
  width: 100%;
  text-align: center;
  vertical-align: top;
  padding-bottom: 3em;
  background-image: url("../../public/both.svg");
  background-repeat: no-repeat;
  background-size: 100% calc(100% - 0.5em);
  background-position: 0 0;
}

div.tree > span > span {
  border: 3px solid #636363;
  border-radius: 50%;
  padding: 3px;
}

/*没有后代的子树不再展示连线和多余空间*/
div.tree > span:only-child {
  background-image: none;
  padding-bottom: 0;
}

</style>