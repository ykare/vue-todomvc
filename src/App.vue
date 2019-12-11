<template>
  <div id="app">
    <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input
          class="new-todo"
          autofocus
          autocomplete="off"
          placeholder="追加する ToDo を入力してください"
          v-model="newTodo"
          @keyup.enter="addTodo"
        />
      </header>
      <!-- この section はデフォルトは非表示で、ToDo が存在するときだけ表示する -->
      <section class="main" v-show="todos.length">
        <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone" />
        <label for="toggle-all">すべてを完了または作業中にします</label>
        <ul class="todo-list">
          <!-- 以下にToDoリストの構造を示す -->
          <!-- リストアイテムの class が `editing` の時は編集中、`completed` の時は完了を示す -->
          <li
            v-for="todo in filteredTodos"
            class="todo"
            :key="todo.id"
            :class="{completed: todo.completed, editing: todo == editedTodo }"
          >
            <div class="view">
              <input :id="'item' + todo.id" class="toggle" type="checkbox" v-model="todo.completed" />
              <label :for="'item' + todo.id" @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input
              class="edit"
              type="text"
              v-model="todo.title"
              v-todo-focus="todo == editedTodo"
              @blur="doneEdit(todo)"
              @keyup.enter="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)"
            />
          </li>
        </ul>
      </section>
      <!-- この footer はデフォルトは非表示で、ToDo が存在するときだけ表示する -->
      <footer class="footer" v-show="todos.length">
        <span class="todo-count">
          <strong>{{filteredTodos.length}}</strong> 件を表示
        </span>
        <ul class="filters">
          <li>
            <a href="#/" :class="{ selected: visibility == 'all' }">すべて</a>
          </li>
          <li>
            <a href="#/active" :class="{ selected: visibility == 'active' }">作業中</a>
          </li>
          <li>
            <a href="#/completed" :class="{ selected: visibility == 'completed' }">完了</a>
          </li>
        </ul>
        <!-- 完了した ToDo がない場合はボタンを非表示にする ↓ -->
        <button
          class="clear-completed"
          @click="removeCompleted"
          v-show="todos.length > remaining"
        >完了したものを削除</button>
      </footer>
    </section>
    <footer class="info">
      <p>ToDo を編集するときはダブルクリックしてください</p>
      <p>Modified by Yoshiyuki Karezaki</p>
      <p>
        Part of
        <a href="http://todomvc.com">TodoMVC</a>
      </p>
    </footer>
  </div>
</template>

<script>
// localStorage persistence
let STORAGE_KEY = "todos-vuejs-2.0";
let todoStorage = {
  fetch: function() {
    let todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    todos.forEach(function(todo, index) {
      todo.id = index;
    });
    todoStorage.uid = todos.length;
    return todos;
  },
  save: function(todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  }
};

// visibility filters
let filters = {
  all: function(todos) {
    return todos;
  },
  active: function(todos) {
    return todos.filter(function(todo) {
      return !todo.completed;
    });
  },
  completed: function(todos) {
    return todos.filter(function(todo) {
      return todo.completed;
    });
  }
};

export default {
  name: "app",
  components: {},
  data: function() {
    return {
      todos: todoStorage.fetch(),
      newTodo: "",
      editedTodo: null,
      visibility: "all"
    };
  },

  // https://jp.vuejs.org/v2/api/#computed
  computed: {
    filteredTodos: function() {
      return filters[this.visibility](this.todos);
    },
    remaining: function() {
      return filters.active(this.todos).length;
    },
    allDone: {
      get: function() {
        return this.remaining === 0;
      },
      set: function(value) {
        this.todos.forEach(function(todo) {
          todo.completed = value;
        });
      }
    }
  },

  // https://jp.vuejs.org/v2/api/#watch
  // localStorage に永続化するため todos を watch
  watch: {
    todos: {
      handler: function(todos) {
        todoStorage.save(todos);
      },
      deep: true
    }
  },

  // https://jp.vuejs.org/v2/api/#mounted
  mounted: function() {
    const app = this;
    // handle routing
    function onHashChange() {
      let visibility = window.location.hash.replace(/#\/?/, "");
      if (filters[visibility]) {
        app.visibility = visibility;
      } else {
        window.location.hash = "";
        app.visibility = "all";
      }
    }

    window.addEventListener("hashchange", onHashChange);
    onHashChange();
  },

  // https://jp.vuejs.org/v2/api/#methods
  methods: {
    addTodo: function() {
      let value = this.newTodo && this.newTodo.trim();
      if (!value) {
        return;
      }
      this.todos.push({
        id: todoStorage.uid++,
        title: value,
        completed: false
      });
      this.newTodo = "";
    },

    removeTodo: function(todo) {
      this.todos.splice(this.todos.indexOf(todo), 1);
    },

    editTodo: function(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },

    doneEdit: function(todo) {
      if (!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.removeTodo(todo);
      }
    },

    cancelEdit: function(todo) {
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    },

    removeCompleted: function() {
      this.todos = filters.active(this.todos);
    }
  },

  // ToDo 編集時に入力フィールドをフォーカスするため
  // カスタムディレクティブを利用
  // https://jp.vuejs.org/v2/guide/custom-directive.html
  directives: {
    "todo-focus": function(el, binding) {
      if (binding.value) {
        el.focus();
      }
    }
  }
};
</script>

<style>
[v-cloak] {
  display: none;
}
</style>
