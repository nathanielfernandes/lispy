<script>
  import { writable } from "svelte/store";
  import defaults from "./data/repl_defaults.json";

  const ERROR = /\{\d+\}>:\s(.*)\s/gimu;

  export let function_signature;
  export let test_cases;
  export let solution;

  let code_editor;
  let editor;

  let state = 0;
  let last_output;
  let last_error;
  let msg;

  $: {
    if (editor !== undefined && editor.getValue() !== function_signature) {
      editor.setValue(function_signature);
      state = 0;
      last_output = undefined;
      last_error = undefined;
    }
  }

  let session_theme;
  let session_fontsize;

  const storedTheme = localStorage.getItem("theme");
  const storedFontSize = localStorage.getItem("fontsize");

  export const theme = writable(storedTheme);
  theme.subscribe((value) => {
    if (value === null) {
      value = "merbivore";
    }
    localStorage.setItem("theme", value);
    session_theme = value;
  });

  export const fontsize = writable(storedFontSize);
  fontsize.subscribe((value) => {
    if (value === null) {
      value = 15;
    }
    localStorage.setItem("fontsize", value);
    session_fontsize = Number(value);
  });

  function setup_ace() {
    editor = ace.edit(code_editor);
    updateTheme();
    updateFontSize();
    editor.session.setMode("ace/mode/lisp");
    editor.resize();
  }

  function updateTheme() {
    theme.set(session_theme);
    editor.setTheme(`ace/theme/${$theme}`);
  }

  function updateFontSize() {
    fontsize.set(session_fontsize);
    editor.setFontSize(session_fontsize);
  }

  function format_code() {
    let user_code = editor.getValue();
    let tests = "\n(deftest test-cases () (check ";
    test_cases.forEach((tc) => (tests += tc + " "));
    return defaults.unit_testing + user_code + tests + "))\n(test-cases)";
  }

  async function run_code() {
    let user_code = format_code();

    const res = await fetch("https://emkc.org/api/v1/piston/execute", {
      method: "POST",
      body: JSON.stringify({ language: "lisp", source: user_code }),
    }).then((res) => res.json());

    state = 1;

    if (res.message === undefined) {
      let error = Array.from(res.output.matchAll(ERROR));
      let is_fatal = res.output.includes("fatal error encountered in SBCL");
      if (error.length > 0) {
        last_output = error[0][1];
        last_error = res.output;
      } else if (is_fatal) {
        last_output = "fatal error encountered in SBCL";
        last_error = res.output;
      } else {
        last_output = res.output.split("|").filter((o) => o != "");
        if (last_output.length === 0) {
          msg = "Fail";
        } else {
          let passed = last_output.filter((o) => o.startsWith("P")).length;

          if (passed < last_output.length) {
            msg = `${passed} / ${last_output.length} Test Cases Passed`;
          } else {
            msg = "All Test Cases Passed :)";
          }
        }

        last_error = undefined;
      }
    } else {
      last_output = [];
      last_error = undefined;
      msg = "Slow Down! You are being Ratelimited";
    }
  }

  function show_solution() {
    state = 2;
  }
</script>

<main>
  <div id="repl">
    <div id="editor-container">
      <div bind:this={code_editor} id="editor">{function_signature}</div>
    </div>

    <div id="output">
      {#if state === 1}
        <code style="color:rgb(254, 99, 48)">Output:</code><br />
        {#if last_error !== undefined}
          <code>{last_output}</code>
          <details>
            <summary>See Full</summary>
            <code>
              <textarea class="solution" spellcheck="false" disabled="true"
                >{last_error}</textarea
              >
            </code>
          </details>
        {:else}
          {#each last_output as out, i}
            {#if out.startsWith("P")}
              <div style="color: green">
                <b>T{i + 1}</b> &nbsp; <code>{out}</code>
              </div>
            {:else}
              <div style="color: red">
                <b>T{i + 1}</b> &nbsp; <code>{out}</code>
              </div>
            {/if}
          {/each}

          <h2>{msg}</h2>
        {/if}
      {:else if state === 2}
        <code style="color:rgb(254, 99, 48)">Solution:</code><br />
        <code>
          <textarea class="solution" spellcheck="false" disabled="true"
            >{solution}</textarea
          >
        </code>
      {/if}
    </div>
  </div>
  <div>
    <button on:click={run_code}>Submit</button>
    <button on:click={show_solution} id="show_solution">Show Solution</button>
    <button on:click={editor.setValue(function_signature)} id="show_solution">
      <!-- svelte-ignore a11y-missing-attribute -->
      <img
        style="width: 30%;"
        src="https://htmlacademy.ru/assets/icons/reload-6x-white.png"
      />
    </button>
  </div>

  <div id="options">
    <span>Theme:</span>
    <select bind:value={session_theme} on:change={updateTheme}>
      {#each defaults.themes as t}
        <option value={t}>
          {t}
        </option>
      {/each}
    </select>
    <br />
    <span>FontSize:</span>
    <input
      type="number"
      bind:value={session_fontsize}
      on:change={updateFontSize}
    />
  </div>
</main>

<svelte:head>
  <script
    on:load={setup_ace}
    src="https://cdnjs.cloudflare.com/ajax/libs/ace/1.4.12/ace.min.js"></script>
</svelte:head>

<style>
  code {
    font-family: "JetBrains Mono", monospace;
  }

  #options {
    margin-top: 0;
  }

  #output {
    /* border-style: solid; */
    /* border-radius: 8px; */
    overflow-y: scroll;
    margin: 0.1rem;
    width: 40%;
    padding: 0.5rem;
    height: 450px;
  }

  #repl {
    display: flex;
    margin: 0.5rem 0;
    margin-bottom: 0;
  }

  #editor-container {
    border-style: solid;
    border-radius: 8px;
    margin: 0.1rem;
    width: 60%;
    height: 450px;
    display: inline-block;
    position: relative;
    overflow: hidden;
  }

  #editor {
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    position: absolute;
  }

  button {
    border: none;
    outline: none;
    padding: 0.5rem 1rem;
    border-radius: 8px;
    background-color: rgb(254, 99, 48);
    color: #fff;
    font-size: 1rem;
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

  #show_solution {
    background-color: rgb(69, 69, 69);
  }

  .solution {
    width: 100%;
    height: 90%;
  }

  @media only screen and (max-width: 1000px) {
    #repl {
      flex-direction: column;
    }

    #editor-container {
      width: 100%;
    }

    #output {
      width: 100%;
    }
  }
</style>
