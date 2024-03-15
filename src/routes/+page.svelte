<script>
  import App from "../components/App.svelte";
  import { draw } from "svelte/transition";
  import { quintOut } from "svelte/easing";
</script>

<main>
  <body>
    <App />
  </body>
</main>

<style>
  body {
    margin: 0;
    padding: 0;
    background-color: rgb(168, 168, 201);
  }
</style>
