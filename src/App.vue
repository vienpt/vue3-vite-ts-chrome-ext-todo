<script setup lang="ts">
import {onMounted, reactive, ref, Ref, toRaw, watch} from "vue";
import IconCheckboxBlankOutline from '~icons/mdi/checkbox-blank-outline';
import IconCheckboxMarked from '~icons/mdi/checkbox-marked';
import IconFileDocumentEdit from '~icons/mdi/file-document-edit';
import IconTrashCan from '~icons/mdi/trash-can';
import draggable from 'vuedraggable'

const dragging = ref(false)
const defaultData = reactive({
  columns: [
    {
      name: 'now',
      list: []
    },
    {
      name: 'new',
      list: []
    },
    {
      name: 'later',
      list: []
    }
  ]
} as { columns: Column[] })


const data = reactive(defaultData)
if (chrome?.storage) {
  chrome.storage.sync.get(['todo-app-data'], function (result) {
    console.log("value key is " + result['todo-app-data']);
    const storeData = result['todo-app-data']
    if (storeData) {
      data.columns = storeData.columns
    }
  })
} else {
  const localStorageString = localStorage.getItem('todo-app-data')
  if (localStorageString) {
    data.columns = JSON.parse(localStorageString).columns
  }
}

// get from localStorage

interface Column {
  name: string;
  list: {
    id: string;
    description: string;
    state: 'new' | 'done';
    editing?: boolean;
  }[];
}

watch(data, (newData) => {
  // chrome.storage.sync.set({ 'todo-app-data': toRaw(data) }, function() {
  //   console.log('value local is set to ', data)
  // })
  if (chrome?.storage) {
    chrome.storage.sync.set({ 'todo-app-data': data }, () => {
      console.log('value local is set to ', data)
    })
  } else {
    localStorage.setItem('todo-app-data', JSON.stringify(newData))
  }
})

const addItem = (event: Event, column: Column) => {
  console.log('add todo', event)
  const formData = new FormData(event.target as HTMLFormElement)
  const description = formData.get('description') as string
  column.list.push({
    id: '5',
    description,
    state: 'new'
  })
}

const editItem = (event: Event, column: Column, listIndex: number) => {
  const form = event.target as HTMLFormElement;
  const formData = new FormData(form);
  const description = formData.get('description') as string;
  console.log(description)

  column.list[listIndex].description = description
  column.list[listIndex].editing = false;
}

const deleteItem = (column: Column, listIndex: number) => {
  column.list.splice(listIndex, 1)
}

onMounted(() => {
  console.log('set debugger')
})
</script>

<template>
  <h2 class="text-red-600 text-xl ">vien test dad hellowork</h2>
  <div class="flex flex-row justify-center space-x-4 pt-10">
    <div
        v-for="column in data.columns"
        :key="column.name"
        class="rounded-md shadow-md bg-slate-100 py-4 px-4 w-1/4"
    >
      <div class="mb-3">{{ column.name }}</div>
      <draggable
          v-model="column.list"
          group="items"
          item-key="id"
          class="space-y-4"
          :class="{'min-h-[40px] bg-slate-200 rounded-md': dragging} "
          ghost-class="ghost"
          @start="dragging = true"
          @end="dragging = false"
      >
        <template #item="{ element: item, index: listIndex }">
          <div
              class="relative group bg-slate-50 rounded-md py-2 px-3 shadow-md flex flex-row space-x-2 items-start"
          >
            <template v-if="item.editing">
              <form @submit.prevent="editItem($event, column, listIndex)" class="flex flex-col w-full">
                <input
                    type="text"
                    name="description"
                    :value="item.description"
                    class="w-full px-3 py-2 rounded-md mb-2"
                />
                <div class="flex justify-between">
                  <button
                      type="button"
                      class="text-xs"
                      @click="item.editing = false"
                  >
                    Cancel
                  </button>
                  <button class="text-xs">Save</button>
                </div>
              </form>
            </template>
            <template v-else>
              <div
                  class="absolute hidden bottom-0 right-0 p-3 bg-slate-50 space-x-3 opacity-90 max-h-full flex-row group-hover:flex">
                <button @click="item.editing = true">
                  <IconFileDocumentEdit/>
                </button>

                <button @click="deleteItem(column, listIndex)">
                  <IconTrashCan class="text-red-700"/>
                </button>
              </div>
              <button class="mt-[3px]" @click="item.state = item.state === 'done' ? 'new' : 'done'">
                <IconCheckboxBlankOutline v-if="item.state !== 'done'"/>
                <IconCheckboxMarked v-else/>
              </button>
              <span
                  class="inline-block break-words min-w-0"
                  :class="{'line-through': item.state === 'done'}"
              >
              {{ item.description }}
            </span>
            </template>
          </div>
        </template>
      </draggable>
      <div class="pt-4">
        <form @submit.prevent="addItem($event, column)" class="flex flex-col items-end">
          <input
              type="text"
              name="description"
              class="w-full px-3 py-2 rounded-md mb-2"
          />
          <button class="float-right">Add</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ghost {
  @apply bg-slate-400 opacity-30;
}
</style>

