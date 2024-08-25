<template>
  <div class="main-wrapper">
    <h1 class="page-title">Vue Your Todo</h1>
    <p class="subtitle">A to-do app powered by vue.js 3!</p>
    <div class="new-task-wrapper">
      <input
        type="text"
        placeholder="Type a new to-do item"
        class="new-task-input"
        v-model="newTaskInput"
        @keyup.enter="addTask"
      />
      <button class="new-task-button" @click="addTask">+ Add</button>
    </div>
    <nav>
      <ul class="tab-wrapper">
        <li class="tab is-active">
          <button class="tab-button" @click="setView('All')">
            All ({{ allTaskLength }})
          </button>
        </li>
        <li class="tab">
          <button class="tab-button" @click="setView('Todos')">
            Pending ({{ currentTaskLength }})
          </button>
        </li>
        <li class="tab">
          <button class="tab-button" @click="setView('Completed')">
            Completed ({{ completedTaskLength }})
          </button>
        </li>
      </ul>
    </nav>
    <ul class="task-list">
      <div
        :style="{ visibility: taskInView.length === 0 ? 'visible' : 'hidden' }"
        class="initial-task-list"
      >
        Add a new task to get started!
      </div>

      <li
        class="task-list-item"
        v-for="taskItem in taskInView"
        :key="taskItem.id"
      >
        <div class="task-list-checkbox-wrapper">
          <IconCheckCircle v-if="taskItem.complete" />
          <IconCircle v-else />
          <input
            type="checkbox"
            class="task-list-checkbox"
            v-model="taskItem.complete"
            :checked="taskItem.complete"
            @change="completeTask(taskItem.id)"
          />
        </div>

        <!-- input text is disabled by default, only enabled when 'edit' is clicked
        line-through and active border style is applied when completed -->
        <input
          type="text"
          v-model="taskItem.label"
          class="task-list-edit-input"
          :disabled="!taskItem.edit"
          :class="{
            complete: taskItem.complete,
            'active-border': taskItem.edit,
          }"
        />

        <div class="task-list-cta">
          <p>
            <!-- edit button is visible when task is not completed, and hidden when completed -->
            <span
              :style="{
                visibility: !taskItem.complete ? 'visible' : 'hidden',
                color: !taskItem.edit ? '#3572EF' : '#399918',
              }"
              style="cursor: pointer; font-weight: bold"
              @click="toggleEdit(taskItem.id)"
            >
              <!-- changes in label depending if it is in edit mode or not -->
              {{ taskItem.edit ? "Save" : "Edit" }}
            </span>
          </p>
          <p>
            <IconDelete
              class="task-list-cta-icon"
              @click="deleteTask(taskItem.id)"
            /><span class="sr-only">Delete</span>
          </p>
        </div>
      </li>
      <div
        :style="{ visibility: taskInView.length !== 0 ? 'visible' : 'hidden' }"
        class="delete-all"
        @click="deleteAll"
      >
        Delete All Recorded Tasks
      </div>
    </ul>
  </div>
</template>

<script>
import { computed, reactive, toRefs, ref } from "vue";
import IconDelete from "./components/IconDelete.vue";
import IconEdit from "./components/IconEdit.vue";
import IconCheckCircle from "./components/IconCheckCircle.vue";
import IconCircle from "./components/IconCircle.vue";
import { v4 as uuid } from "uuid";

export default {
  name: "App",
  components: {
    IconDelete,
    IconEdit,
    IconCheckCircle,
    IconCircle,
  },
  setup() {
    const state = reactive({
      currentView: "All",
      newTaskInput: "",
      taskList: JSON.parse(localStorage.getItem("taskList")) || [],
    });

    // for viewing and filtering of tasklist depending on status
    const taskList = reactive({
      all: computed(() => state.taskList),
      current: computed(() =>
        state.taskList.filter((item) => item.complete === false)
      ),
      completed: computed(() =>
        state.taskList.filter((item) => item.complete === true)
      ),
    });

    // determining the number or task on each status
    const taskViews = reactive({
      allTaskLength: computed(() => taskList.all.length),
      currentTaskLength: computed(() => taskList.current.length),
      completedTaskLength: computed(() => taskList.completed.length),
    });

    const taskInView = computed(() => {
      if (state.currentView === "All") {
        // for sorting the completed task and placing it on the bottom of the list
        return state.taskList.sort((a, b) => a.complete - b.complete);
      } else if (state.currentView === "Todos") {
        // will show not completed tasks only
        return state.taskList.filter((item) => item.complete === false);
      } else if (state.currentView === "Completed") {
        // will show completed tasks only
        return state.taskList.filter((item) => item.complete === true);
      } else {
        return state.taskList;
      }
    });

    // initial state when a task is added
    const addTask = () => {
      state.taskList.push({
        id: uuid(),
        label: state.newTaskInput,
        edit: false,
        complete: false,
      });
      state.newTaskInput = "";
      state.isDisabled = true;
      updateLocalStorage();
    };

    // boolean is set to true to disable input text edit mode when a task is added
    const isDisabled = ref(true);

    // toggles edit mode for a task
    const toggleEdit = (taskId) => {
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList[taskIndex].edit = !state.taskList[taskIndex].edit;
      if (!state.taskList[taskIndex].edit) {
        state.isDisabled = !state.isDisabled;
      }
      updateLocalStorage();
    };

    // completed task status toggle
    const completeTask = (taskId) => {
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList[taskIndex.complete] = !state.taskList[taskIndex.complete];
      updateLocalStorage();
    };

    // delete a task from the list with splice on a certain taskIndex
    const deleteTask = (taskId) => {
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList.splice(taskIndex, 1);
      updateLocalStorage();
    };

    const deleteAll = () => {
      if (state.taskList.length > 0) {
        state.taskList = [];
        updateLocalStorage();
      }
    };

    // to view all, pending or completed tasks
    const setView = (viewLabel) => {
      updateLocalStorage();
      state.currentView = viewLabel;
    };

    // to update the local storage with the updated task list
    // also to prevent losing the data when the page is refreshed
    const updateLocalStorage = () => {
      localStorage.setItem("taskList", JSON.stringify(state.taskList));
    };

    return {
      ...toRefs(state),
      ...toRefs(taskViews),
      addTask,
      deleteTask,
      deleteAll,
      completeTask,
      setView,
      toggleEdit,
      taskInView,
      isDisabled,
    };
  },
};
</script>

<style scoped>
.initial-task-list {
  font-family: Arial, Helvetica, sans-serif;
  font-style: italic;
  font-weight: bold !important;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 240px;
  width: 100%;
  color: rgb(163, 161, 161);
  position: absolute;
}
.delete-all {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 13px;
  font-weight: bold !important;
  color: #c7253e;
  cursor: pointer;
}
.task-list-checkbox-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}
.task-list-checkbox {
  position: absolute;
  opacity: 0;
  left: -3px;
  bottom: 2px;
}
.task-list-item {
  border: 1px solid #f6f6f6;
  box-shadow: 2px 2px 8px 0px #c0c0c0;
  border-radius: 8px;
  display: flex;
  align-items: center;
  padding: 0px 16px;
  margin-bottom: 16px;
}
.task-list-item:focus,
.task-list-item:hover {
  border: 1px solid #0631f8;
}
.task-list-cta {
  display: flex;
  column-gap: 16px;
}
.task-list {
  padding: 0;
}
.tab:hover .tab-button {
  color: #0631f8;
}
.task-list-edit-input,
.task-list-text {
  margin: 0 12px;
  font-weight: bold;
  flex: 1;
  border: 0;
  font-size: 16px;
  min-width: 50px;
}

.task-list-edit-input:disabled {
  background: transparent;
}
.tab-wrapper {
  display: flex;
  column-gap: 20px;
  list-style: none;
  margin: 20px 0px 0px 0px;
  padding: 0;
}

.tab {
  padding-bottom: 6px;
}
.tab-button {
  border: 0;
  background-color: transparent;
  color: #6b6b6b;
  letter-spacing: 1px;
  font-weight: bold;
  padding: 0;
}
.tab-button:hover {
  cursor: pointer;
}
.page-title {
  font-size: 40px;
  letter-spacing: 1px;
  color: #2d2d2d;
  margin-bottom: 0;
  margin-top: 104px;
}
.subtitle {
  color: #6b6b6b;
  margin-top: 0;
}
.main-wrapper {
  max-width: 600px;
  margin: 0 auto;
  text-align: left;
  position: relative;
}
html {
  background-color: #fbfbff;
}
.new-task-input {
  padding: 16px;
  flex: 1;
  color: #959595;
  border: none;
  border-top-left-radius: 7px;
  border-bottom-left-radius: 7px;
  box-shadow: 2px 2px 8px 0px #c0c0c0;
}
.new-task-wrapper {
  display: flex;
}
.new-task-button {
  padding: 18px 24px;
  background-color: #0631f8;
  color: white;
  border: none;
  border-top-right-radius: 7px;
  border-bottom-right-radius: 7px;
}
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}
.task-list-cta-icon {
  cursor: pointer;
}
.is-active {
  color: #0631f8;
}
.complete {
  text-decoration: line-through;
}
.active-border {
  box-shadow: inset 2px 2px 5px rgba(0, 0, 0, 0.2);
  border-radius: 5px;
  padding: 7px 9px;
}
</style>
