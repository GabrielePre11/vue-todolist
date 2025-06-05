<script setup lang="ts">
import { ref, computed, onMounted, watch } from "vue";
import Container from "./Container.vue";

//============ ToDoItem INTERFACE ============//
interface ToDoItem {
  id: number;
  label: string;
  isCompleted: boolean;
}

//============ Filter TYPE ============//
type Filter = "All" | "Completed";

//============ REFS ============//
const taskInput = ref<string>("");
const todoList = ref<ToDoItem[]>([]);

// Count the total number of completed tasks
const completedTasks = computed(
  () => todoList.value.filter((task) => task.isCompleted).length
);

const activeCategory = ref<Filter>("All");

//============ CATEGORIES ============//
const categories: Filter[] = ["All", "Completed"];

//============ ADD TASK FUNCTION ============//
const addTask = () => {
  if (taskInput.value.trim() === "") return;

  const newTask: ToDoItem = {
    id: Date.now(),
    label: taskInput.value.trim(),
    isCompleted: false,
  };

  todoList.value = [...todoList.value, newTask];
  taskInput.value = "";
};

//============ COMPLETE TASK  FUNCTION ============//
const completeTask = (id: number) => {
  const checkedTask = todoList.value.map((task) =>
    task.id === id ? { ...task, isCompleted: !task.isCompleted } : task
  );
  todoList.value = checkedTask;
};

//============ DELETE TASK FUNCTION ============//
const deleteTask = (id: number) => {
  todoList.value = todoList.value.filter((task) => task.id !== id);
};

//============ TASK FILTERING ============//
const filterTasks = computed(() => {
  if (activeCategory.value === "Completed") {
    return todoList.value.filter((task) => task.isCompleted);
  } else {
    return todoList.value;
  }
});

//============ LOCAL STORAGE ============//
/*
◉ onMounted: it's equal to the hook useEffect() (or componentOnMount method of class components) in React! It only runs the code once, when the component it's mounted.

◉ watch: it's equal to the hook useEffect(), but with dependencies, thus it will run the code when the dependences change.

◉ { deep: true }: It's necessary when watching reactive arrays or objects, to detect nested changes as well.
*/

onMounted(() => {
  const savedTasks = localStorage.getItem("todoList");

  if (savedTasks) {
    todoList.value = JSON.parse(savedTasks);
  }
});

watch(
  todoList,
  (newTasks) => {
    localStorage.setItem("todoList", JSON.stringify(newTasks));
  },
  { deep: true }
);
</script>

<template>
  <!--============ HEADER ============ -->
  <header class="header">
    <h1 class="header-title">
      <span class="vue-span">Vue</span>
      To-Do List
    </h1>
  </header>

  <!--============ CONTAINER ============ -->
  <Container>
    <div class="input-wrapper">
      <!--============ INPUT ============ -->
      <input
        type="text"
        placeholder="Buy spaghetti and tomato sauce (not with ketchup)..."
        class="task-input"
        v-model="taskInput"
        @keyup.enter="addTask"
      />

      <!--============ ADD BUTTON ============ -->
      <button class="add-button" @click="addTask">Add</button>
    </div>

    <!--============ FILTERS & INFO ============ -->
    <div class="task-info">
      <!--============ FILTERS ============ -->
      <div class="filters">
        <button
          v-for="category in categories"
          :key="category"
          class="filter-button"
          :class="{ active: activeCategory === category }"
          @click="activeCategory = category"
        >
          {{ category }}
        </button>
      </div>

      <!--============ INFO ============ -->
      <div class="info">
        <h3 class="task-info-title">
          Tasks: <span class="info-number">{{ todoList.length }}</span>
        </h3>

        <h3 class="task-info-title">
          Completed Task:
          <span class="info-number">{{ completedTasks }}</span>
        </h3>
      </div>
    </div>

    <!--============ TO-DO LIST ============ -->
    <ul class="todo-list">
      <!--============ TO-DO ITEM ============ -->
      <li
        class="todo-item"
        v-for="task in filterTasks"
        :key="task.id"
        @click="completeTask(task.id)"
      >
        <div class="todo-item-wrapper">
          <!--============ CHECK TASK BUTTON ============ -->
          <button
            class="icon check-icon"
            :class="{ completed: task.isCompleted }"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="size-6"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M9 12.75 11.25 15 15 9.75M21 12c0 1.268-.63 2.39-1.593 3.068a3.745 3.745 0 0 1-1.043 3.296 3.745 3.745 0 0 1-3.296 1.043A3.745 3.745 0 0 1 12 21c-1.268 0-2.39-.63-3.068-1.593a3.746 3.746 0 0 1-3.296-1.043 3.745 3.745 0 0 1-1.043-3.296A3.745 3.745 0 0 1 3 12c0-1.268.63-2.39 1.593-3.068a3.745 3.745 0 0 1 1.043-3.296 3.746 3.746 0 0 1 3.296-1.043A3.746 3.746 0 0 1 12 3c1.268 0 2.39.63 3.068 1.593a3.746 3.746 0 0 1 3.296 1.043 3.746 3.746 0 0 1 1.043 3.296A3.745 3.745 0 0 1 21 12Z"
              />
            </svg>
          </button>

          <!--============ LABEL ============ -->
          <p class="task-text" :class="{ completed: task.isCompleted }">
            {{ task.label }}
          </p>
        </div>

        <!--============ DELETE TASK BUTTON ============ -->
        <button class="icon delete-icon" @click="deleteTask(task.id)">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="size-6"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0"
            />
          </svg>
        </button>
      </li>

      <!--============ NO TAKS ALERT ============ -->
      <!--============ NO TASKS ============ -->
      <p
        v-if="filterTasks.length === 0 && activeCategory !== 'Completed'"
        class="no-tasks"
      >
        There are no tasks yet!
      </p>

      <!--============ NO COMPLETED TASKS ============ -->
      <p
        v-if="!completedTasks && activeCategory === 'Completed'"
        class="no-completed-tasks"
      >
        There are no completed tasks yet!
      </p>
    </ul>
  </Container>
</template>

<style scoped>
/*============ HEADER ============*/
.header {
  position: fixed;
  display: flex;
  align-items: center;
  justify-content: center;
  top: 0;
  left: 0;
  width: 100%;
  padding-block: 20px;
  color: #dfd0b8;
  background-color: var(--blue-col);
  font-family: sans-serif;
}

.header-title {
  display: flex;
  align-items: center;
  font-size: 30px;
  gap: 15px;
  font-family: var(--secondary-font);
}

.vue-span {
  color: var(--vue-green);
  font-weight: 700;
}

/*============ TASKS INFO ============*/
.task-info {
  display: flex;
  align-items: center;
  margin-block: 15px;
  justify-content: space-between;
}

.info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.info-number {
  background-color: var(--blue-col);
  border-radius: 50%;
  padding: 8px 15px;
}

.task-info-title {
  display: flex;
  align-items: center;
  font-family: var(--secondary-font);
  gap: 10px;
  width: max-content;
  font-size: 18px;
  color: white;
}

/*============ FILTERS ============*/
.filters {
  display: flex;
  align-items: center;
  gap: 15px;
}

.filter-button {
  padding: 10px;
  transition: color 0.3s;
  border-radius: 10px;
  background-color: var(--blue-col);
  color: #fff;
  font-size: 16px;
}

.filter-button.active {
  border: 2px solid var(--vue-green);
}

.filter-button:is(:hover, :focus) {
  background-color: #1b212b;
}

/*============ TASK INPUT, TASK AND TO-DO ITEM ============*/
.task-input {
  border: 2px solid var(--blue-col);
  flex: 1;
  padding: 15px 30px;
  border-radius: 18px;
  font-size: 1.1rem;
  font-family: var(--primary-font);
}

.todo-list {
  display: grid;
  gap: 10px;
  margin-block: 25px;
}

.input-wrapper {
  display: flex;
  align-items: center;
  gap: 5px;
}

.todo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: var(--blue-col);
  color: gray;
  padding: 10px;
  font-size: 1rem;
  font-family: sans-serif;
  border-radius: 10px;
  cursor: pointer;
  transition: color 0.5s;
}

.todo-item:is(:hover, :focus) {
  background-color: #1b212b;
}

.todo-item-wrapper {
  display: flex;
  align-items: center;
  gap: 15px;
}

.task-text {
  font-size: 18px;
  color: #dfd0b8;
}

.task-text.completed {
  text-decoration: line-through;
}

.no-completed-tasks,
.no-tasks {
  margin-block-start: 30px;
  font-size: 22px;
  text-align: center;
  font-family: var(--primary-font);
  color: #dfd0b8;
}

/*============ TASK ICONS ============*/
.icon {
  width: 35px;
  transition: color 0.3s;
  background: none;
}

.check-icon {
  color: #dfd0b8;
}

.check-icon:is(:hover, :focus),
.check-icon.completed {
  color: var(--vue-green);
}

.delete-icon {
  color: red;
}

/*============ TASK ADD BUTTON ============*/
.add-button {
  padding: 15px 30px;
  transition: color 0.4s;
  border-radius: 16px;
  background-color: var(--blue-col);
  color: #fff;
  font-size: 20px;
  font-family: var(--primary-font);
}

.add-button:is(:hover, :focus) {
  background-color: #1b212b;
}

/*============ MEDIA QUERY ============*/
@media (max-width: 320px) {
  .info {
    flex-direction: column;
  }
}

@media (max-width: 510px) {
  .task-info {
    flex-direction: column;
    gap: 18px;
  }
}

@media (max-width: 450px) {
  .input-wrapper {
    display: flex;
    flex-direction: column;
  }

  .task-input {
    width: 100%;
  }

  .add-button {
    width: 100%;
  }
}
</style>
