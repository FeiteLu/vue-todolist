<template>
  <div class="todolist">
    <div>
      <h1>Todo List</h1>
      <form @submit.prevent="addTodo">
        <input v-model="newTodo" placeholder="Add new todo" />
        <button>Add</button>
      </form>
    </div>
    <div class="todos">
      <div class="todo">
        <h2>Pending Tasks</h2>
        <ul>
          <li v-for="todo in pendingTodos" :key="todo.id">
            <span @click="updateTodoStatus(todo)" :class="{ 'completed': todo.completed }">{{ todo.text }}</span>
            <button @click="removeTodo(todo)">Delete</button>
          </li>
        </ul>
      </div>
      <div class="todo">
        <h2>Completed Tasks</h2>
        <ul>
          <li v-for="todo in completedTodos" :key="todo.id">
            <span @click="updateTodoStatus(todo)" :class="{ 'completed': todo.completed }">{{ todo.text }}</span>
            <button @click="removeTodo(todo)">Delete</button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { computed } from '@vue/reactivity';
import { onMounted, ref } from 'vue';
import { getTodos, createTodo, deleteTodo, updateTodo } from '../apis/TodoApis'

export default {
  setup() {
    const newTodo = ref('');
    const todos = ref([]);

    const loadTodos = async () => {
      const data = await getTodos();
      todos.value = data;
    }

    const addTodo = async () => {
      if (newTodo.value.trim() === '') return;

      const todo = await createTodo({
        id: todos.value.length + 1,
        text: newTodo.value,
        completed: false
      })

      todos.value.push(todo);
      newTodo.value = '';
    };

    const removeTodo = async (todo) => {
      await deleteTodo(todo.id);
      todos.value.splice(todos.value.indexOf(todo), 1)
    };

    const updateTodoStatus = async (todo) => {
      await updateTodo(todo.id, { completed: !todo.completed })
      todo.completed = !todo.completed;
    }

    onMounted(() => {
      loadTodos();
    })

    const pendingTodos = computed(() => todos.value.filter(todo => !todo.completed));
    const completedTodos = computed(() => todos.value.filter(todo => todo.completed));

    return {
      newTodo,
      addTodo,
      removeTodo,
      updateTodoStatus,
      pendingTodos,
      completedTodos
    };
  },
};
</script>

<style></style>



<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.todos {
  display: flex;
  justify-content: center;

}

.todo {
  display: flex;
  flex-direction: column;
  padding: 10px;
  padding-top: 0;
  margin: 20px;
}

h2 {
  margin: 40px 0 0;
}

ul {
  display: flex;
  flex-direction: column;
  justify-content: center;
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 10px 10px;
}

.completed {
  text-decoration: line-through;
}
</style>
