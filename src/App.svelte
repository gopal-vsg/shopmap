<script lang="ts">
  import Counter from "./lib/Counter.svelte";
</script>

<main>
  <div class="card">
    <Counter />
  </div>

  <div class="flex flex-row">
    <div class="flex flex-col">
      <div>One
        <button class="btn btn-primary">Click me</button>
        <button class="btn btn-secondary">Click me</button>
        <button class="btn btn-tertiary">Click me</button>
        <button class="btn btn-sm btn-tertiary">Click me</button>
      </div>
    </div>
    <div class="flex flex-col">
      <div>Three</div>
      <div>Four</div>
      <div>Five</div>
    </div>
  </div>
</main>

<style>
</style>
