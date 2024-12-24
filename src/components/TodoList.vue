<template>

  <div>

    <h1>Todo app</h1>

    <input
      class="task-input"
      v-model="newTodo" 
      placeholder="Новая задача" 
      @keyup.enter="addTodo"
    />

    <button @click="addTodo">Добавить</button>

    <button 
      class="del-all" 
      @click="removeCompleted">Удалить выполненные
    </button>

    <button 
      @click="checkAll"
      class="del-all check-all"
    >Выполнить все</button>

    <ul>
      <div v-for="todo in todos" :key="todo.documentId">

        <input 
          class="checkbox"
          type="checkbox" 
          v-model="todo.completed"
          @click="checkTodo(todo.documentId, todo.completed)"
        />

        <span>{{ todo.text }}</span>
        <button 
          @click="removeTodo(todo.documentId)" 
          class="btn"
        >X</button>
      </div>
    </ul>

  </div>

</template>


<script setup lang="ts">

import { ref, onMounted } from 'vue';
import axios from 'axios';

interface Todo {
  documentId: string;
  text: string;
  completed: boolean;
}

const todos = ref<Todo[]>([]); 

const newTodo = ref('');

const URL = 'http://localhost:1337/api/tasks';

onMounted(async () => {
  try {
    const response = await axios.get(URL);
    console.log('Данные из Strapi:', response.data);
    todos.value = response.data.data.map((item: Todo) => ({
      documentId: item.documentId,
      text: item.text,
      completed: item.completed
    }));
  } catch (error) {
    console.error('Ошибка при получении данных из Strapi:', error);
  }
});

const addTodo = async () => {
  if (newTodo.value.trim()) {
    try {
      const response = await axios.post(URL, {
        data: {
          text: newTodo.value,
          completed: false
        }
      });
      console.log('Задача успешно добавлена:', response.data);
      const newTask = response.data.data;
      todos.value = [...todos.value, { documentId: newTask.documentId, text: newTask.text, completed: newTask.completed }];
      newTodo.value = '';
    } catch (error) {
      console.error('Ошибка при добавлении задачи:', error);
    }
  }
};

const removeTodo = async (documentId:string) => {
  await axios.delete(`${URL}/${documentId}`)
  todos.value = todos.value.filter((todo)=> todo.documentId !== documentId)
}

const removeCompleted = async () => {
  const filtered = todos.value.filter((todo) => todo.completed);
  filtered.forEach((task)=> removeTodo(task.documentId))
}

const checkTodo = async (documentId:string, completed: boolean) => {
  console.log(completed)
  if (!completed) {
    await axios.put(`${URL}/${documentId}`, {
      data: {
        completed: true
      }
    })
  } else {
    await axios.put(`${URL}/${documentId}`, {
      data: {
        completed: false
      }
    })
  }
}

const checkAll = async () => {
  todos.value = todos.value.map((todo) => ({
    ...todo,
    completed: true,
  }));

  for (const todo of todos.value) {
    try {
      await axios.put(`${URL}/${todo.documentId}`, {
        data: {
          completed: true,
        },
      });
    } catch (error) {
      console.error(`Ошибка при обновлении задачи ${todo.documentId}:`, error);
    }
  }
};

</script>



<style scoped>

div {
  font-family: Arial, sans-serif;
  max-width: 700px;
  margin: 20px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

ul div {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
  margin-bottom: 10px;
  padding: 10px 15px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

ul div:hover {
  transform: translateY(-2px);
}

span {
  flex: 1;
  font-size: 16px;
  color: #555;
  cursor: pointer;
}

input:checked + span {
  text-decoration: line-through;
  color: #aaa;
}

button.btn {
  background-color: #ff4d4d;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 6px 12px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button.btn:hover {
  background-color: #e60000;
}


button {
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  cursor: pointer;
  transition: background-color 0.3s;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

button:hover {
  background-color: #45a049;
}

.task-input {
  width: calc(100% - 288px);
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin-right: 10px;
  font-size: 14px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

input[type="checkbox"] {
  width: 16px;
  height: 16px;
  margin-right: 10px;
  cursor: pointer;
  margin-bottom: 4px;
}

input[type="checkbox"]:checked {
  accent-color: #4caf50;
}

.del-all {
  margin-left: 10px;
  background-color: #ff4d4d;
}

.del-all:hover {
  background-color: #e60000;      
}

.check-all {
  margin-left: calc(100% - 120px);
  margin-top: 8px;
}

</style>