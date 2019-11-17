<script>
  import PouchDB from "pouchdb-browser";
  export let name;
  let db = new PouchDB("test");
  console.log(db);
</script>

<style>

</style>

<main>
  <h1>Hello {name}!</h1>
</main>
