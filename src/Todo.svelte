<script>
  import { onMount } from "svelte";
  import { writable } from "svelte/store";

  const todos = writable([]);
  let newTodo = "";

  onMount(async () => {
    const response = await fetch("http://localhost:3333/todos");
    const data = await response.json();
    todos.set(data);
  });

  async function addTodo() {
    const todo = { text: newTodo, done: false };
    const response = await fetch("http://localhost:3333/todos", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(todo),
    });
    const data = await response.json();
    todos.update((n) => [...n, data]);
    newTodo = "";
  }

  async function deleteTodo(id) {
    await fetch(`http://localhost:3333/todos/${id}`, {
      method: "DELETE",
    });
    todos.update((n) => n.filter(todo => todo.id !== id));
  }

  async function toggleDone(todo) {
    const updatedTodo = { ...todo, done: !todo.done };
    const response = await fetch(`http://localhost:3333/todos/${todo.id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(updatedTodo),
    });
    const data = await response.json();
    todos.update((n) => n.map(t => t.id === data.id ? data : t));
  }

  async function updateTodoText(todo, text) {
    const updatedTodo = { ...todo, text };
    const response = await fetch(`http://localhost:3333/todos/${todo.id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(updatedTodo),
    });
    const data = await response.json();
    todos.update((n) => n.map(t => t.id === data.id ? data : t));
  }
</script>

<style>
  .container {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
  }
  input {
    padding: 10px;
    font-size: 1rem;
  }
  button {
    padding: 10px;
    font-size: 1rem;
    margin-left: 10px;
  }
  ul {
    list-style: none;
    padding: 0;
  }
  li {
    padding: 10px;
    border-bottom: 1px solid #ccc;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  li.done {
    text-decoration: line-through;
  }
  .todo-text {
    flex-grow: 1;
    margin-right: 10px;
  }
  .edit-input {
    flex-grow: 1;
  }
</style>

<div class="container">
  <h1>To-Do List</h1>
  <input bind:value={newTodo} placeholder="Add a new task" />
  <button on:click={addTodo}>Add</button>
  <ul>
    {#each $todos as todo}
      <li class:done={todo.done}>
        <div class="todo-text" on:dblclick={() => (todo.editing = true)}>{todo.text}</div>
        {#if todo.editing}
          <input class="edit-input" type="text" bind:value={todo.text} on:blur={() => {updateTodoText(todo, todo.text); todo.editing = false;}} on:keydown={(e) => {if (e.key === 'Enter') {updateTodoText(todo, todo.text); todo.editing = false;}}} />
        {/if}
        <button on:click={() => toggleDone(todo)}>{todo.done ? "Undone" : "Done"}</button>
        <button on:click={() => deleteTodo(todo.id)}>Delete</button>
      </li>
    {/each}
  </ul>
</div>
