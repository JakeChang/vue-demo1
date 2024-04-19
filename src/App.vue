<template>
  <div class="container mx-auto">
    <div class="flex justify-center mt-4 gap-2">
      <input type="text" placeholder="今天要做什麼？" class="input input-bordered input-md w-full max-w-xs" v-model="text" />
      <button class="btn" @click="save" :disabled="isSave || text === ''">
        <span class="loading loading-spinner loading-xs" v-if="isSave"></span>
        儲存
      </button>
    </div>

    <div class="overflow-x-auto">
      <table class="table table-zebra">
        <thead>
          <tr>
            <th>編號</th>
            <th>內容</th>
            <th>#</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in items" :key="index">
            <th>{{ index + 1 }}</th>
            <td>
              <p v-if="!item.isEdit">{{ item.text }}</p>
              <input type="text" placeholder="今天要做什麼？" class="input input-bordered input-md w-full max-w-xs"
                v-model="item.text" v-else />
            </td>
            <td>
              <button class="btn btn-xs" @click="changeMode(index)" v-if="!item.isEdit">編輯</button>
              <div class="flex gap-2" v-if="item.isEdit">
                <button class="btn btn-success btn-outline btn-xs" @click="update(index)">更新</button>
                <button class="btn btn-error btn-outline btn-xs" @click="remove(index)">刪除</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { Parse } from 'parse/dist/parse.min.js';

export default {
  setup() {
    const text = ref('');
    const isSave = ref(false);
    const items = ref([]);

    async function getData() {
      try {
        const query = new Parse.Query("ToDo");
        let datas = await query.find();

        items.value = [];
        datas.forEach(data => {
          items.value.push({
            "id": data.id,
            "text": data.get('text'),
          })
        });
      }
      catch (error) {
        console.log(error);
      }
    }

    async function save() {
      isSave.value = true;

      const toDoList = new Parse.Object("ToDo");
      toDoList.set("text", text.value);
      try {
        await toDoList.save()
        text.value = "";
        await getData();
        isSave.value = false;
      }
      catch (error) {
        alert('上傳失敗 ' + error.message);
      }
    }

    function changeMode(index) {
      items.value[index].isEdit = true;
    }

    async function update(index) {
      let item = items.value[index];

      try {
        const query = new Parse.Query("ToDo");

        query.equalTo("objectId", item.id);
        const toDo = await query.first();
        toDo.set('text', item.text);
        await toDo.save()
        items.value[index].isEdit = false;
      }
      catch (error) {
        alert('更新失敗 ' + error.message);
      }
    }

    async function remove(index) {
      let item = items.value[index];

      try {
        const query = new Parse.Query("ToDo");
        query.equalTo("objectId", item.id);
        const toDo = await query.first();
        await toDo.destroy();
        await getData();
      }
      catch (error) {
        console.log(error);
      }
    }

    onMounted(() => {
      getData();
    });

    return {
      text, isSave, items,
      save, changeMode, update, remove,
    }
  },
}
</script>