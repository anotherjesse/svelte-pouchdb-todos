<script>
  import PouchDB from "pouchdb-browser";
  import { quintOut } from "svelte/easing";
  import { crossfade } from "svelte/transition";
  import { flip } from "svelte/animate";
  import { onMount } from "svelte";

  var db = new PouchDB("todos");
  var remoteCouch = false;

  const loadTodos = () => {
    db.allDocs({ include_docs: true, descending: true }, function(err, doc) {
      todos = doc.rows.map(r => r.doc);
    });
  };

  db.changes({
    since: "now",
    live: true
  }).on("change", loadTodos);

  const [send, receive] = crossfade({
    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === "none" ? "" : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
      };
    }
  });

  let todos = [];
  onMount(loadTodos);

  function add(input) {
    db.post({
      done: false,
      description: input.value
    });

    input.value = "";
  }

  const toggleComplete = todo => {
    todo.done = !todo.done;
    db.put(todo);
  };

  function remove(todo) {
    db.remove(todo);
  }
</script>

<style>
  .new-todo {
    font-size: 1.4em;
    width: 100%;
    margin: 2em 0 1em 0;
  }

  .board {
    max-width: 36em;
    margin: 0 auto;
  }

  .left,
  .right {
    float: left;
    width: 50%;
    padding: 0 1em 0 0;
    box-sizing: border-box;
  }

  h2 {
    font-size: 2em;
    font-weight: 200;
    user-select: none;
  }

  label {
    top: 0;
    left: 0;
    display: block;
    font-size: 1em;
    line-height: 1;
    padding: 0.5em;
    margin: 0 auto 0.5em auto;
    border-radius: 2px;
    background-color: #eee;
    user-select: none;
  }

  input {
    margin: 0;
  }

  .right label {
    background-color: rgb(180, 240, 100);
  }

  button {
    float: right;
    height: 1em;
    box-sizing: border-box;
    padding: 0 0.5em;
    line-height: 1;
    background-color: transparent;
    border: none;
    color: rgb(170, 30, 30);
    opacity: 0;
    transition: opacity 0.2s;
  }

  label:hover button {
    opacity: 1;
  }
</style>

<div class="board">
  <input
    class="new-todo"
    placeholder="what needs to be done?"
    on:keydown={event => event.which === 13 && add(event.target)} />

  <div class="left">
    <h2>todo</h2>
    {#each todos.filter(t => !t.done) as todo (todo._id)}
      <label
        in:receive={{ key: todo._id }}
        out:send={{ key: todo._id }}
        animate:flip>
        <input
          type="checkbox"
          checked={todo.done}
          on:change={() => toggleComplete(todo)} />
        {todo.description}
        <button on:click={() => remove(todo)}>x</button>
      </label>
    {/each}
  </div>

  <div class="right">
    <h2>done</h2>
    {#each todos.filter(t => t.done) as todo (todo._id)}
      <label
        in:receive={{ key: todo._id }}
        out:send={{ key: todo._id }}
        animate:flip>
        <input
          type="checkbox"
          checked={todo.done}
          on:change={() => toggleComplete(todo)} />
        {todo.description}
        <button on:click={() => remove(todo)}>x</button>
      </label>
    {/each}
  </div>
</div>
