<script context="module">
  import { Marquee } from "@selemondev/svelte-marquee";
  import "@selemondev/svelte-marquee/dist/style.css";
  import { onMount } from "svelte";
</script>

<script>
  export let conf;
  let isReady = false;
  let data = {};
  onMount(async () => {
    await initTmi();
    const response = await fetch(`${conf.api}/api/hanabii/all/hanabii77`, {
      method: "GET",
      headers: {
        "Content-Type": "application/json"
      }
    });
    const result = await response.json();
    data = result.data;
    isReady = true;
  });
  const update = async () => {
    const response = await fetch(`${conf.api}/api/hanabii/all/hanabii77`, {
      method: "GET",
      headers: {
        "Content-Type": "application/json"
      }
    });
    const result = await response.json();
    data = result.data;
  };
  const initTmi = async () => {
    const client = new tmi.Client({
      options: { messagesLogLevel: "info" },
      connection: {
        reconnect: true,
        secure: true
      },
      identity: {
        username: "justinfan123456",
        password: ""
      },
      channels: [conf.bot]
    });

    client.on("message", async (target, context, msg, self) => {
      if (context.username === conf.bot && msg.split(" ")[0].toLowerCase() === "!update") {
        await update();
      }
    });
    await client.connect().catch(console.error);
  };
  const text1 = "謝謝";
  const text2 = "斗內";
  const subText = "份訂閱";
  const cheerText = "個小奇點";

  const numberBig = ["０", "１", "２", "３", "４", "５", "６", "７", "８", "９"];
</script>

{#if isReady}
  <div style="display: flex; --duration: {(Object.entries(data).length + 1) * 10}s;">
    <Marquee direction="up" innerClassName="content">
      <div style="height: 640px"></div>
      {#each "歡迎諸位善信大德光臨哈娜比七七實況台".split("") as char}
        <div>{char}</div>
      {/each}
      <div style="height: 60px"></div>
      {#each Object.entries(data) as [key, data]}
        {#each text1.split("") as char}
          <div>{char}</div>
        {/each}
        {#each data.name.split("") as char}
          <div>{char}</div>
        {/each}
        {#each text2.split("") as char}
          <div>{char}</div>
        {/each}
        {#each data.count.toString().split("") as char}
          <div>{numberBig[char]}</div>
        {/each}
        {#if data.type === "c"}
          {#each cheerText.split("") as char}
            <div>{char}</div>
          {/each}
        {/if}
        {#if data.type === "s"}
          {#each subText.split("") as char}
            <div>{char}</div>
          {/each}
        {/if}
        <div>！</div>
        <div style="height: 60px"></div>
      {/each}
    </Marquee>
  </div>
{/if}

<style lang="scss">
  :global(.content) {
    width: 70px;
    font-family: "標楷體";
    font-weight: bold;
    color: #f9c80e;
    font-size: 64px;
    line-height: 76px;
    display: flex;
    flex-direction: column;
    gap: 1px;
    align-items: center;
    justify-content: center;
    text-shadow:
      0 0 5px #000000,
      0 0 5px #000000,
      0 0 5px #000000,
      0 0 5px #000000;
  }
</style>
