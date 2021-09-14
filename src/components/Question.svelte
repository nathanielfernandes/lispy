<script>
  import Repl from "./Repl.svelte";

  export let question_number;
  export let name;
  export let description;
  export let examples;
  export let function_signature;
  export let solution;
  export let test_cases;
</script>

<main>
  <div class="question">
    <div class="top">
      <h5 class="number">Question: {question_number}</h5>
      <h3 class="name">{name}</h3>
      <p class="desc">
        {description}
      </p>
      <code class="examples">
        {#each examples as ex}
          {ex} <br />
        {/each}
      </code>
    </div>

    <div class="code">
      <Repl {function_signature} {test_cases} {solution} />
    </div>
  </div>
</main>

<style>
  .question .top {
    width: 60%;
  }
  .question .code {
    width: 100%;
  }

  .question .name {
    margin-top: 0;
    color: rgb(254, 99, 48);
  }

  .question .number {
    margin: 0.2rem 0 0 0;
  }

  .question .examples {
    font-family: "JetBrains Mono", monospace;
  }
</style>
