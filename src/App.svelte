<script lang="ts">
  // import svelteLogo from "./assets/svelte.svg";
  // import Counter from "./lib/Counter.svelte";
  import { getMessages, type Message } from "codicent-js";

  interface FormInfo {
    name: string;
    content: string;
  }

  let token = new URLSearchParams(window.location.search).get("token") || "";
  let debug = "";
  let messages: Message[] = [];
  let selectedForm: string;
  let project = "uxreview";

  const getForms = (messages: Message[]): FormInfo[] => {
    return messages
      .map((m) => m.content.match(/#form\s+(.+)\s*\n+([\w\W]+)/))
      .filter((m) => m)
      .map((m) => ({ name: m[1], content: m[2] } as FormInfo));
  };

  $: {
    if (token) {
      getMessages(token, "feedback")
        .then((msgs) => msgs.filter((m) => m.content.includes("#form ")))
        .then((msgs) => {
          messages = msgs;
          console.log(msgs);
          // debug = JSON.stringify(msgs, null, 2);
          debug = JSON.stringify(getForms(msgs), null, 2);
        });
    }
  }

  $: templateUrl = `https://codicent.com/form/${project}/${selectedForm}?token=${token}`;
</script>

<main>
  <h1>Codicent Form Builder</h1>

  <div>
    <label for="token">Codicent API token</label>
    <input id="token" type="text" placeholder="Enter Codicent API token" bind:value={token} />
  </div>
  <div>
    <label for="forms">Choose form:</label>
    <select name="forms" id="forms" bind:value={selectedForm}>
      {#each getForms(messages) as form}
        <option value={form.name}>{form.name}</option>
      {/each}
    </select>
  </div>
  <!-- <div>
    <pre>{debug}</pre>
  </div> -->
  <div>
    <iframe title="form" src={templateUrl} width="320" height="640" />
  </div>

  <!-- <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank" rel="noreferrer">SvelteKit</a>, the
    official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">Click on the Vite and Svelte logos to learn more</p> -->
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
