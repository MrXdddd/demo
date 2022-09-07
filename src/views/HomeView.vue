<template>
  <div class="main">
    <h1 style="text-align: center">{{ msg }}</h1>
    <br />
    <div class="add">
      <Input v-model="todoItem" placeholder="请填写任务" style="width: 300px" />
      <Button @click="add" style="height: 30px" type="primary">ADD</Button>
      <Button @click="search" style="height: 30px" icon="ios-search">
        Search
      </Button>
      <Button @click="reset" style="height: 30px" icon="ios-refresh"
        >Reset</Button
      >
    </div>
    <div class="content">
      <Table
        border
        :columns="columns"
        :data="todoList"
        :key="Math.random()"
        width="'100%'"
      >
        <template slot-scope="{ row, index }" slot="value">
          <span v-if="!row.editTable" @dblclick="edit(row, index)"
            >{{ row.value }}
          </span>
          <Input
            ref="ref_input"
            id="deviceInput"
            v-model="row.value"
            placeholder="Enter something..."
            style="width: 100%"
            v-else
            @on-blur="lostForce(row, index)"
            @on-enter="lostForce(row, index)"
          />
        </template>
        <template slot-scope="{ row, index }" slot="action">
          <Button
            type="primary"
            size="small"
            style="margin-right: 5px"
            @click="edit(row)"
            v-if="!row.editTable"
            >Edit</Button
          >
          <Button
            type="error"
            size="small"
            @click="del(index)"
            style="margin-right: 5px"
            >Delete</Button
          >
          <Button
            type="success"
            size="small"
            @click="finish(index)"
            v-if="!row.finished"
            >Submit</Button
          >
        </template>
      </Table>
    </div>
  </div>
</template>

<script lang='ts'>
import {
  reactive,
  watchEffect,
  ref,
  watch,
  onMounted,
  toRefs,
  defineComponent,
} from "@vue/composition-api";

interface column {
  title?: string;
  key?: string;
  slot?: string;
  align?: string;
  render?: (h: any, params: { [key: string]: any }) => {};
}

export default defineComponent({
  name: "WatchEffect",

  setup() {
    let msg: string = "todoList";
    //表头
    let columns: column[] = reactive([]);
    columns.push(
      {
        title: "编号",
        key: "id",
        align: "center",
      },
      {
        title: "工作内容",
        key: "value",
        slot: "value",
        align: "center",
      },
      {
        title: "状态",
        slot: "finished",
        key: "finished",
        align: "center",
        render: (h, params) => {
          let flag = params.row.finished;
          if (flag == true) {
            flag = "已完成";
          } else {
            flag = "未完成";
          }
          return h(
            "div",
            {
              style: {
                display: "inline-block",
                padding: "0 7px",
                height: "24px",
                fontSize: "14px",
                borderRadius: "3px",
                background: params.row.finished == true ? "#36c447" : "red",
                color: "white",
              },
            },
            flag
          );
        },
      },
      {
        title: "操作",
        slot: "action",
        align: "center",
      }
    );

    //工作内容
    let todoItem = ref<string>("");

    //生成id
    let itemId = ref<number>(1);

    //工作表单
    let state: { todoList: any } = reactive({
      todoList: [],
    });
    let resetList: {}[] = [];

    //判断是否为查询状态
    let isSearch: boolean = false;

    //查询结果
    let searchList: {}[] = reactive([]);

    //获取元素
    const ref_input: any = ref(null);

    //添加
    let add = () => {
      state.todoList.push({
        value: todoItem.value,
        finished: false,
        editTable: false,
        id: itemId.value,
      });

      if (isSearch == false) {
        resetList = state.todoList;
      } else {
        resetList.push({
          value: todoItem.value,
          finished: false,
          editTable: false,
          id: itemId.value,
        });
        console.log(resetList);
      }
      todoItem.value = "";
      itemId.value++;
    };

    //修改
    const edit = (
      item: {
        [key: string]: any;
      },
      index: any
    ) => {
      item.editTable = true;
      //自动获取焦点
      setTimeout(() => {
        ref_input.value.focus();
      });
    };

    //删除
    const del = (index: number) => {
      state.todoList.splice(index, 1);
      resetList = state.todoList;
    };

    //查询
    const search = () => {
      isSearch = true;
      searchList = state.todoList.filter((item: any) => {
        return item.value.indexOf(todoItem.value) != -1;
      });
      state.todoList = searchList;
    };

    //重置
    const reset = () => {
      isSearch = false;
      state.todoList = resetList;
      todoItem.value = "";
    };

    //是否完成
    const finish = (index: number) => {
      state.todoList[index].finished = !state.todoList[index].finished;
    };

    //失去焦点触发
    const lostForce = (item: { [key: string]: any }, index: number) => {
      item.editTable = false;
    };

    return {
      msg,
      todoItem,
      ...toRefs(state),
      itemId,
      columns,
      ref_input,
      reset,
      add,
      finish,
      edit,
      lostForce,
      search,
      del,
      searchList,
    };
  },
});
</script>

<style lang="less" scoped>
.main {
  width: 100%;
  height: 100%;
  .add {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .content {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    width: 70%;
    margin: 10px auto;
    padding: 0 auto;
  }
}
.add /deep/ .ivu-btn {
  margin-left: 10px;
}
/deep/ .ivu-table-wrapper {
  width: 100% !important;
}
</style>