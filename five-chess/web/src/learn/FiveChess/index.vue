<template>
  <h2>Five Chess</h2>
  <h3>You are {{myColor}}</h3>
  <div class="flex">
    <div class="chessboard" :style="{ width: width + 'px' }">
      <Cell v-for="(row, index) in rows" :key="index" @click="click(row, 1)">
        <Piece :color="row.color"></Piece>
      </Cell>
    </div>
  </div>

  <div class="tool">
    <button @click="reset">reset</button>
    <!-- <input class="rank-int" type="text" v-model="rank" /> -->
  </div>
</template>

<script setup>
import {
  ref,
  reactive,
  getCurrentInstance,
  nextTick,
  watch,
  onMounted,
  computed,
} from "vue";
import Cell from "./Cell.vue";
import Piece from "./Piece.vue";

const rank = ref(13);

let rows = ref(
  new Array(rank.value * rank.value).fill(null).map((item, index) => {
    return {
      color: "",
      coord: [index % rank.value, Math.floor(index / rank.value)],
      index: index,
    };
  })
);

let count = ref(0); // 步数
let isWin = false;
let myColor = ref('');
const click = async (row, type) => {
  if (row.color != "") return;
  if (type == 1 && currentColor.value != myColor.value) return;
  console.log('myColor.value: ', myColor.value);
  console.log('currentColor: ', currentColor);
  row.color = count.value % 2 == 0 ? "black" : "white";
  count.value++;

  //判断胜利条件
  judge(row);

  nextStep_myself(row);
};

let continuous;

const judge = (row) => {
  console.log("row: ", row);
  let i = 0;
  let continuous = 0;
  // 行胜利 , 定位到落子的那一行
  for (
    i = row.coord[1] * rank.value;
    i < (row.coord[1] + 1) * rank.value;
    i++
  ) {
    if (rows.value[i].color === row.color) {
      continuous++;
    } else {
      continuous = 0;
    }

    // 为什么无法等到DOM刷新后才弹出？
    if (continuous === 5) {
      if (!isWin) {
        isWin = true;
        alert("You win!");
      }
    }
  }
  continuous = 0;
  // 列胜利，定位到那一列
  for (i = row.coord[0]; i < rank.value * rank.value; i += rank.value) {
    if (rows.value[i].color == row.color) {
      continuous++;
    } else {
      continuous = 0;
    }
    // 为什么无法等到DOM刷新后才弹出？
    if (continuous === 5) {
      if (!isWin) {
        isWin = true;
        alert("Win!");
      }
    }
  }

  // 左上到右下
  continuous = 0;
  for (
    i = row.index - (rank.value + 1) * Math.min(row.coord[0], row.coord[1]);
    i < rank.value * rank.value;
    i += rank.value + 1
  ) {
    if (rows.value[i].color == row.color) {
      continuous++;
    } else {
      continuous = 0;
    }
    // 为什么无法等到DOM刷新后才弹出？
    if (continuous === 5) {
      if (!isWin) {
        isWin = true;
        alert("Win!");
      }
    }
  }

  // 左上到右下
  continuous = 0;
  for (
    i = row.index - (rank.value - 1) * Math.min(row.coord[0], row.coord[1]);
    i < rank.value * rank.value;
    i += rank.value - 1
  ) {
    if (rows.value[i].color == row.color) {
      continuous++;
    } else {
      continuous = 0;
    }
    // 为什么无法等到DOM刷新后才弹出？
    if (continuous === 5) {
      if (!isWin) {
        isWin = true;
        alert("row.color is Win!");
      }
    }
  }
};

const reset = () => {
  for (let i = 0; i < rows.value.length; i++) {
    rows.value[i].color = "";
  }
  isWin = false;
};

let width = ref(rank.value * 42);
watch(rank, (newRank, oldRank) => {
  if (newRank > 0 && newRank < 9) {
    rows.value = reactive(
      new Array(newRank * newRank).fill(null).map((item, index) => {
        return {
          color: "",
          coord: [index % rank.value, Math.floor(index / rank.value)],
        };
      })
    );
    width.value = newRank * 42;
  }
});
var ws = null;
// 对战室http://192.168.2.75/
onMounted(() => {
  ws = new WebSocket("ws://192.168.2.75:3000");
  ws.onmessage = nextStep_others;
  ws.onopen = function () {
    //当WebSocket创建成功时，触发onopen事件
    // console.log("open");

    // 第一次应该注册一下棋子的颜色
    ws.send("hello"); //将消息发送到服务端
  };
});

const nextStep_others = (evt) => {
  console.log('evt: ', evt);
  let data = JSON.parse(evt.data);
  console.log('data: ', data);

  if (data?.first) {
    myColor.value = data.first;
    console.log('myColor: ', myColor);
  }

  try {
    var step = JSON.parse(JSON.parse(evt.data));
    console.log("step111: ", step);
  } catch (error) {}

  if (step && step.color) {
    if(step.color == myColor.value){
      return
    }
    rows.value[step.index] = step;
    count.value ++
    judge(step);
  }
};

const nextStep_myself = (row) => {
  ws.send(JSON.stringify(row));
};

// let currentColor = '123'
let currentColor = computed(() => {
  return count.value % 2 === 0 ? "black" : "white";
});
</script>
<style scoped>
.chessboard {
  margin: 20px auto;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  cursor: pointer;
  /* width: 126px; */
}

.flex {
  display: flex;
}

.tool {
  align-items: center;
  display: flex;
  flex-direction: column;
}
.rank-int {
  width: 30px;
  margin-top: 7px;
}
</style>
