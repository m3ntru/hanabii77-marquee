<script context="module">
  import { onMount } from "svelte";
  import Display from "../component/display.svelte";
  import Control from "../component/control.svelte";
</script>

<script>
  let page = "";
  let target = "hanabii77";
  let bot = "hanabii88";
  let api = "https://m3ntru-api.vercel.app";
  onMount(async () => {
    page = new URLSearchParams(window.location.search).get("page");
  });
</script>

<title>hanabii77-marquee-{page}</title>
{#if page === "display"}
  <Display conf={{ target, bot, api }} />
{/if}

{#if page === "control"}
  <Control conf={{ target, bot, api }} />
{/if}
