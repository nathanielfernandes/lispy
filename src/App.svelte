<script>
  import { writable } from "svelte/store";

  import questions from "./components/data/questions.json";
  import Question from "./components/Question.svelte";
  import Nav from "./components/Nav.svelte";
  // import Code from "./components/Code.svelte";

  let session_question;

  const storedQuestionNum = localStorage.getItem("question");
  export const questionNum = writable(storedQuestionNum);
  questionNum.subscribe((value) => {
    if (value === null) {
      value = 0;
    }
    localStorage.setItem("question", value);
    session_question = Number(value);
  });

  let data = questions[session_question];
  data["question_number"] = session_question + 1;

  function refresh() {
    data = questions[session_question];
    data["question_number"] = session_question + 1;
    questionNum.set(session_question);
  }
  function updateQuestion(i) {
    session_question = i + session_question;
    refresh();
  }

  function prevQuestion() {
    if (session_question > 0) {
      updateQuestion(-1);
    }
  }

  function nextQuestion() {
    if (session_question < questions.length - 1) {
      updateQuestion(1);
    }
  }
</script>

<main>
  <Nav />
  <button on:click={prevQuestion}>Prev Question</button>
  <button on:click={nextQuestion}>Next Question</button>
  <span id="jump">Jump:</span>
  <select bind:value={session_question} on:change={refresh} id="jump">
    {#each questions as q, i}
      <option value={i}>{i + 1}) {q.name}</option>
    {/each}
  </select>
  <Question {...data} />
</main>

<svelte:head>
  <link
    href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@700&display=swap"
  />
  <script
    src="https://cdnjs.cloudflare.com/ajax/libs/ace/1.4.12/ace.js"></script>
</svelte:head>

<style>
  main {
    /* text-align: center; */
    padding: 1em;
    max-width: 500px;
    margin: 0 auto;
  }

  #jump {
    font-size: 0.8rem;
  }

  /* button:hover {
    background-color: rgb(254, 118, 72);
  } */

  /* .emoji {
    margin-top: 0;
    font-size: 10rem;
  } */

  button {
    border: none;
    outline: none;
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    background-color: rgb(254, 99, 48);
    color: #fff;
    font-size: 0.7rem;
    font-weight: 600;
    cursor: pointer;
    box-shadow: rgba(0, 9, 61, 0.2) 0px 4px 0px;
    transition: 0.2s;
  }

  button:active {
    transform: translateY(3px);
    background-color: rgb(253, 88, 32);
    box-shadow: rgba(0, 9, 61, 0.2) 0px 1px 0px;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
