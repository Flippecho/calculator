<template>
  <div class="calculator">
    <span class="equation" style="grid-area: expression">{{ equation }}</span>
    <span class="result" style="grid-area: result">{{ result }}</span>

    <button style="grid-area: ac" @click="clear">AC</button>
    <button style="grid-area: delete" @click="deleteLast">DEL</button>
    <button style="grid-area: equal" @click="calculate">=</button>

    <button style="grid-area: add" @click="append('+')">+</button>
    <button style="grid-area: sub" @click="append('-')">-</button>
    <button style="grid-area: mul" @click="append('×')">×</button>
    <button style="grid-area: div" @click="append('÷')">÷</button>
    <button style="grid-area: dot" @click="append('.')">.</button>
    <button style="grid-area: pre-bracket" @click="append('(')">(</button>
    <button style="grid-area: post-bracket" @click="append(')')">)</button>

    <button style="grid-area: number-1" @click="append('1')">1</button>
    <button style="grid-area: number-2" @click="append('2')">2</button>
    <button style="grid-area: number-3" @click="append('3')">3</button>
    <button style="grid-area: number-4" @click="append('4')">4</button>
    <button style="grid-area: number-5" @click="append('5')">5</button>
    <button style="grid-area: number-6" @click="append('6')">6</button>
    <button style="grid-area: number-7" @click="append('7')">7</button>
    <button style="grid-area: number-8" @click="append('8')">8</button>
    <button style="grid-area: number-9" @click="append('9')">9</button>
    <button style="grid-area: number-0" @click="append('0')">0</button>
  </div>
</template>

<script>
import Decimal from "decimal.js";

export default {
  name: "TheCalculator",
  mounted() {
    const _this = this;
    document.addEventListener("keydown", function (e) {
      if (e.shiftKey && (e.key === 'Backspace' || e.key === 'Delete')) _this.clear();
      else if (e.key === '*') _this.append('×');
      else if (e.key === '/') _this.append('÷');
      else if (_this.isValidKey(e.key) || _this.isOperand(e.key)) _this.append(e.key);
      else if (e.key === 'Delete' || e.key === 'Backspace') _this.deleteLast();
      else if (e.key === '=' || e.key === 'Enter') _this.calculate();
    });
  },
  props:{
    postOrderExpression: {
      type: Array
    }
  },
  data() {
    return {
      equation: '',
      lastChar: '',
      countLeftBrackets: 0,
      countRightBrackets: 0,
      isDecimalAdded: false,
      importantPosition: [],
      result: '0',
      expressionTree: ''
    }
  },
  watch: {
    postOrderExpression(newExp) {
      this.result = this.evaluate(newExp);
    }
  },
  methods: {
    TNode(op1, op2, opr) {
      this.op1 = op1;
      this.op2 = op2;
      this.opr = opr;
    },
    isOperator(character) {
      return ['+', '-', '×', '÷'].indexOf(character) > -1
    },
    isOperand(character) {
      return character.match(/[0-9]/);
    },
    isValidKey(character) {
      return ['+', '-', '(', ')', '.'].indexOf(character) > -1;
    },
    append(character) {
      this.result = '0';
      if (character === '.') {
        if (this.isDecimalAdded) this.result = '每个操作数至多含一个小数点';
        else if ((this.lastChar === '' || this.isOperand(this.lastChar))) {
          if (this.lastChar === '') this.equation = '0';
          this.equation += character;
          this.lastChar = character;
          this.isDecimalAdded = true;
        } else this.result = this.lastChar + ' 后不能是 ' + character;
      } else if (character === '(') {
        if (this.lastChar === '' || this.lastChar === '(' || this.isOperator(this.lastChar)) {
          this.equation += character;
          this.lastChar = character;
          this.countLeftBrackets++;
        } else this.result = this.lastChar + ' 后不能是 ' + character;
      } else if (character === ')') {
        if (this.countRightBrackets >= this.countLeftBrackets) this.result = "括号不匹配！";
        else if (this.lastChar === ')' || this.isOperand(this.lastChar)) {
          this.equation += character;
          this.lastChar = character;
          this.countRightBrackets++;
        } else this.result = this.lastChar + ' 后不能是 ' + character;
      } else if (this.isOperator(character)) {
        if (this.lastChar === '' || this.lastChar === ')' || this.isOperand(this.lastChar)) {
          if (this.lastChar === '') this.equation = 0;
          this.equation += character;
          this.lastChar = character;
          this.isOperatorAdded = true;
          if (this.isDecimalAdded === true) {
            this.isDecimalAdded = false;
            this.importantPosition.push(this.equation.length);
          }
        } else this.result = this.lastChar + ' 后不能是 ' + character;
      } else if (this.isOperand(character)) {
        if (this.lastChar === '' || this.lastChar === '.' || this.lastChar === '(' || this.isOperand(this.lastChar) || this.isOperator(this.lastChar)) {
          this.equation += character;
          this.lastChar = character;
          this.isOperatorAdded = false;
        } else this.result = this.lastChar + ' 后不能是 ' + character;
      }
    },
    clear() {
      this.equation = ''
      this.lastChar = ''
      this.countLeftBrackets = 0
      this.countRightBrackets = 0
      this.isDecimalAdded = false
      this.importantPosition = []
      this.result = '0'
      this.resultGraph = []
      this.expressionTree = {}
      this.$emit("expressionTreeChanged",this.expressionTree);
      this.preOrderExpression = []
      this.inOrderExpression = []
      console.clear();
    },
    deleteLast() {
      this.result = '0';
      let last = this.equation.charAt(this.equation.length - 1);
      let last2 = this.equation.charAt(this.equation.length - 2);
      if (last === '.') this.isDecimalAdded = false;
      else if (this.isOperator(last)) {
        this.isOperatorAdded = false;
        let temp = this.importantPosition.pop();
        if (this.equation.length === temp) this.isDecimalAdded = true;
        else this.importantPosition.push(temp);
      } else if (last === '(') this.countLeftBrackets--;
      else if (last === ')') this.countRightBrackets--;
      else if (this.isOperator(last2)) this.isOperatorAdded = true;
      this.equation = this.equation.slice(0, -1);
      this.lastChar = this.equation.charAt(this.equation.length - 1);
    },
    calculate() {
      if (this.lastChar === '') {
        this.clear();
        this.result = '请输入表达式';
      }
      else if (this.countLeftBrackets > this.countRightBrackets) this.result = '括号不匹配';
      else if (this.lastChar === '.') this.result = '请输入小数部分';
      else if (this.isOperator(this.lastChar)) this.result = '请输入下一个操作数';
      else {
        console.clear();
        let expression = this.equation.replace(new RegExp('×', 'g'), '*')
            .replace(new RegExp('÷', 'g'), '/')
            .replace(/\s*/g, "");
        this.expressionTree = this.getExpressionTree(expression);
        console.log('Expression Tree: ')
        console.log(this.expressionTree);
        this.$emit("expressionTreeChanged",this.expressionTree);
      }
    },
    getExpressionTree(expression) {
      const pLen = expression.length
      // 初始化两个栈
      const stack1 = [] // 运算符栈
      const stack2 = [] // 操作数栈
      const OP_REG = /\d|\./ // 操作数 正则表达式
      const OPERATOR_REG = /^\+|-|\*|\/$/ // 运算符 正则表达式

      // 从左至右扫描中缀表达式
      for (let i = 0; i < pLen; i++) {
        const cur = expression[i];
        // 是符号数则判断是否大于栈顶符号
        if (OPERATOR_REG.test(cur)) {
          // 栈中没有符号 当前符号是左括号 栈顶符号是左括号 当前符号大于栈顶符号
          if (stack1.length === 0 || stack1[stack1.length - 1] === '(' || getOperatorLevel(stack1[stack1.length - 1]) < getOperatorLevel(cur)) {
            stack1.push(cur);
            console.log("operator " + cur + " pushed");
          }
          // 当前符号小于栈顶符号，生成新的树节点，并重新判断当前位置
          else {
            let op1 = stack2.pop(), op2 = stack2.pop(), opr = stack1.pop();
            stack2.push(new this.TNode(op2, op1, opr));
            i--;
            console.log("New TNode Created");
          }
        }
        // 为右括号时不断生成新的树节点直到匹配到左括号
        else if (cur === ')') {
          while (stack1.length > 0 && stack1[stack1.length - 1] !== '(') {
            let op1 = stack2.pop(), op2 = stack2.pop(), opr = stack1.pop();
            stack2.push(new this.TNode(op2, op1, opr));
            console.log("New TNode Created");
          }
          stack1.pop();
          console.log("left bracket popped");
        } else if (cur === '(') {
          stack1.push(cur);
          console.log("operator " + cur + " pushed");
        }
        // 是操作数则存入 stack2
        else {
          let j = i;
          let str = '';
          while (OP_REG.test(expression[j])) str += expression[j++];
          i = j - 1;
          stack2.push(str);
          console.log("number " + str + " pushed");
        }
      }

      // 不断生成新的树节点直到符号栈为空，操作数栈中的即为表达式二叉树
      while (stack1.length >= 1) {
        let op1 = stack2.pop(), op2 = stack2.pop(), opr = stack1.pop();
        stack2.push(new this.TNode(op2, op1, opr));
        console.log("New TNode Created");
      }

      console.log(stack2);
      return stack2.pop();

      /**
       * @description 获取运算符的等级
       * @params {string} operator 运算符
       * */
      function getOperatorLevel(operator) {
        const LOW_LEVEL_OPERATOR_REG = /^\+|-$/ // 低级运算符 正则表达式
        const HIGH_LEVEL_OPERATOR_REG = /^\+|-|\*|\/$/ // 高级操作符 正则表达式
        if (LOW_LEVEL_OPERATOR_REG.test(operator)) {
          return 1
        } else if (HIGH_LEVEL_OPERATOR_REG.test(operator)) {
          return 2
        }
      }
    },
    evaluate(expression) {
      const stack = [];
      const OP_REG = /^-?\d+\.?\d*$/ // 操作数 正则表达式
      const OPERATOR_REG = /^\+|-|\*|\/$/ // 运算符 正则表达式
      for (let i = 0; i < expression.length; i++) {
        let cur = expression[i];
        if (OP_REG.test(cur)) {
          stack.push(parseFloat(cur));
        } else if (OPERATOR_REG.test(cur)) {
          const num2 = stack.pop();
          const num1 = stack.pop();
          if (cur === '+') {
            stack.push(Decimal.add(num1, num2));
          } else if (cur === '-') {
            stack.push(Decimal.sub(num1, num2));
          } else if (cur === '*') {
            stack.push(Decimal.mul(num1, num2));
          } else if (cur === '/') {
            stack.push(Decimal.div(num1, num2));
          }
        }
      }
      return stack.pop();
    }
  }
}

</script>

<style scoped>

.calculator {
  --button-width: 90px;
  --button-height: 70px;
  --display-height: 80px;

  width: calc(var(--button-width) * 5);
  height: calc(var(--display-height) * 2 + var(--button-height) * 4);
  position: absolute;
  float: left;
  top: 15vh;
  left: 5vw;

  display: grid;
  grid-template-columns: repeat(5, var(--button-width));
  grid-template-rows: repeat(2, var(--display-height)), repeat(4, var(--button-height));
  grid-template-areas:
  "expression expression expression expression expression"
  "result result result result result"
  "number-7 number-8 number-9 pre-bracket post-bracket"
  "number-4 number-5 number-6 add sub"
  "number-1 number-2 number-3 mul div"
  "number-0 dot delete ac equal";

  border-radius: 11px;
  background: linear-gradient(145deg, #ffffff, #e6e6e6);
  box-shadow: 11px 11px 30px #c9c9c9, -11px -11px 30px #ffffff;
}

.calculator button {
  margin: 1px;
  padding: 0;
  border: 0;
  display: block;
  outline: none;
  font-size: 21px;
  font-family: Helvetica, serif;
  font-weight: normal;
  border-radius: 1px;
  background: #fafafa;
}

.calculator button:active {
  box-shadow: -4px -4px 10px -8px rgba(255, 255, 255, 1) inset, 4px 4px 10px -8px rgba(0, 0, 0, .3) inset;
}

.calculator .equation {
  text-align: left;
  height: 80px;
  line-height: var(--display-height);
  font-size: 30px;
  font-family: Helvetica, serif;
  padding: 0 20px;
  white-space: nowrap;
  overflow-x: auto;
}

.calculator .result {
  text-align: right;
  height: 80px;
  line-height: var(--display-height);
  font-size: 30px;
  font-family: Helvetica, serif;
  padding: 0 20px;
  white-space: nowrap;
  overflow-x: auto;
}

</style>