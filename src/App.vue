<script setup lang="ts">
import category_data from '@/data/categories.json';
import task_data from '@/data/tasks.json';
import {ref,computed} from 'vue';
import type {Category,Task,CategoryTask} from './types/';
import CategoryItems from '@/components/CategoryItem.vue';
import AddCategory from '@/components/AddCategory.vue';

const categories = ref<Category[]>(category_data);
const tasks = ref<Task[]>(task_data);
const dragTask = ref<Task | null>(null);

const renderCategoryTask = computed<CategoryTask[]>(()=> {
  return categories.value.map(category => {
    const filterTasks = tasks.value.filter(
      task => task.category_id === category.id
    );
    return {
      id:category.id,
      name:category.name,
      tasks:filterTasks
    };
  });
})

const setDragTask = (task:Task)=> {
  dragTask.value = task;
}

const dragOverTask = (overTask:Task)=> {
  if (dragTask.value?.id !== overTask.id) {
    const deleteIndex = tasks.value.findIndex(
      (task) => task.id === dragTask.value?.id
    );
    const addIndex = tasks.value.findIndex((task) => task.id === overTask.id);
    if (dragTask.value !== null) {
      tasks.value.splice(deleteIndex, 1);
      dragTask.value.category_id = overTask.category_id; //追加
      tasks.value.splice(addIndex, 0, dragTask.value);
    }
  }
}

const dragOverCategory = (categoryTask: CategoryTask) => {
  if (dragTask.value?.category_id !== categoryTask.id) {
    const filterTasks = tasks.value.filter(
      (task) => task.category_id === categoryTask.id
    );
    if (filterTasks.length === 0 && dragTask.value !== null)
      dragTask.value.category_id = categoryTask.id;
  }
};

</script>

<template>
 <div class="h-12 p-2">
    <h1 class="text-sm font-bold">Trello風タスク管理</h1>
    <div class="flex">
      <CategoryItems
        class="min-w-[400px]"
        v-for="categoryTask in renderCategoryTask"
        :key="categoryTask.id"
        :categoryTask="categoryTask"
        @setDragTask="setDragTask"
        @dragOverTask="dragOverTask"
        @dragover="dragOverCategory(categoryTask)"
      />
      <AddCategory/>
    </div>
 </div>
</template>
