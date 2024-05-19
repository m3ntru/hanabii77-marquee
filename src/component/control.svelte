<script context="module">
  import { onMount } from "svelte";
  import tmi from "tmi.js";
  import SolidButton from "./SolidButton.svelte";
  import moment from "moment";
</script>

<script>
  export let conf;

  const status = ["Loading...", "Success", "Fail"];
  let tmiStatus = 0;
  let dataStatus = 0;
  let list = [];
  let all = {};
  let allJson = "";
  let jsonStatus = "";
  let client = null;
  let name = "";
  let display = "";
  let type = "s";
  let count = "0";
  let sending = false;
  let success = "";
  let paramsToken = "";

  onMount(async () => {
    moment.locale("zh-tw");
    paramsToken = new URLSearchParams(window.location.search).get("api");
    try {
      await getAll();
      await getList();
      dataStatus = 1;
    } catch (error) {
      dataStatus = 2;
    }
    await initTmi();
  });

  const getAll = async () => {
    const response = await fetch(`${conf.api}/api/hanabii/all/hanabii77`, {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${paramsToken}`
      }
    });
    const result = await response.json();
    all = result.data;
    allJson = JSON.stringify(all, undefined, 4);
  };

  const getList = async () => {
    const response = await fetch(`${conf.api}/api/hanabii/list/hanabii77`, {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${paramsToken}`
      }
    });
    const result = await response.json();
    list = result;
  };

  const postList = async (data) => {
    const response = await fetch(`${conf.api}/api/hanabii/list/hanabii77`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${paramsToken}`
      },
      body: JSON.stringify(data)
    });
    return await response.json();
  };

  const deleteList = async (id) => {
    const response = await fetch(`${conf.api}/api/hanabii/list/${id}`, {
      method: "DELETE",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${paramsToken}`
      }
    });
    return await response.json();
  };

  const postAll = async (data) => {
    const response = await fetch(`${conf.api}/api/hanabii/all/hanabii77`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${paramsToken}`
      },
      body: JSON.stringify({
        twitch: conf.target,
        data
      })
    });
    return await response.json();
  };

  const getText = (data) => {
    const subText = "份訂閱";
    const cheerText = "個小奇點";
    return `謝謝 ${data.name} 斗內 ${data.count} ${data.type === "s" ? subText : cheerText}`;
  };

  const initTmi = async () => {
    const paramsUser = new URLSearchParams(window.location.search).get("user");
    const paramsKey = new URLSearchParams(window.location.search).get("token");
    client = new tmi.Client({
      options: { messagesLogLevel: "info" },
      connection: {
        reconnect: true,
        secure: true
      },
      identity: {
        username: paramsUser ? paramsUser : "justinfan123456",
        password: paramsKey ? paramsKey : ""
      },
      channels: [conf.target]
    });
    client.on("connected", (address, port) => {
      tmiStatus = 1;
    });
    client.on(
      "submysterygift",
      (channel, username, numbOfSubs, methods, userstate, senderCount) => {
        addSub(username, userstate.login, numbOfSubs);
      }
    );
    client.on("cheer", (channel, userstate, message) => {
      addCheer(userstate["display-name"], userstate["username"], userstate["bits"]);
    });
    await client.connect().catch(console.error);
  };
  const addCheer = async (display, name, count) => {
    sending = true;
    success = "";
    try {
      let item = {
        type: "c",
        name,
        display,
        count
      };
      const keyName = name + "_c";
      let temp = all;
      if (temp[keyName]) {
        temp[keyName].count += count;
      } else {
        temp[keyName] = { type: "c", name: display, count: count };
      }
      all = temp;
      allJson = JSON.stringify(all, undefined, 4);
      let result = await postList({
        item,
        all: {
          twitch: conf.target,
          data: all
        }
      });
      item._id = result.insertedId;
      item.createdAt = new Date().toISOString();
      list = [item, ...list];

      success = "s";
    } catch {
      success = "f";
    }
    sending = false;
    client.say(conf.bot, "!update");
  };
  const addSub = async (display, name, count) => {
    sending = true;
    success = "";
    try {
      let item = {
        type: "s",
        name,
        display,
        count
      };
      const keyName = name + "_s";
      let temp = all;
      if (temp[keyName]) {
        temp[keyName].count += count;
      } else {
        temp[keyName] = { type: "s", name: display, count: count };
      }
      all = temp;
      allJson = JSON.stringify(all, undefined, 4);
      let result = await postList({
        item,
        all: {
          twitch: conf.target,
          data: all
        }
      });
      item._id = result.insertedId;
      item.createdAt = new Date().toISOString();
      list = [item, ...list];
      success = "s";
    } catch {
      success = "f";
    }
    sending = false;
    client.say(conf.bot, "!update");
  };

  const send = () => {
    const countInt = parseInt(count);
    if (name === "" || !countInt) return;
    if (type === "c") {
      addCheer(display || name, name, countInt);
    }
    if (type === "s") {
      addSub(display || name, name, countInt);
    }
  };

  const deleteItem = async (id) => {
    await deleteList(id);
    let newList = list.filter((a) => a._id !== id);
    list = newList;
  };

  const updateJson = async () => {
    jsonStatus = "";
    let newAll = null;
    try {
      newAll = JSON.parse(allJson);
    } catch {
      jsonStatus = "格式有誤";
      return;
    }
    try {
      all = newAll;
      await postAll(newAll);
      jsonStatus = "上傳成功";
      client.say(conf.bot, "!update");
    } catch {
      jsonStatus = "上傳失敗";
      return;
    }
  };
</script>

<div class="control-bg">
  {#if dataStatus === 1 && tmiStatus === 1}
    <div class="control-body">
      <div class="control-title">手動加入</div>
      <div
        class="card"
        style="display: flex; flex-direction: column; align-items:center; gap: 8px;"
      >
        <table class="control-table">
          <tr style="height: 40px"
            ><td style="width: 120px;">英文ID Name</td><td><input bind:value={name} /></td></tr
          >
          <tr style="height: 40px"
            ><td style="width: 120px;">中文ID Display</td><td><input bind:value={display} /></td
            ></tr
          >
          <tr style="height: 40px"
            ><td style="width: 120px;">種類 Type</td><td
              style="display: flex; padding:10px; justify-content: space-between; gap: 8px;"
              ><label style="display: flex; gap: 4px;">
                <input
                  checked={type === "s"}
                  on:change={(event) => {
                    type = event.currentTarget.value;
                  }}
                  type="radio"
                  name="type"
                  value="s"
                /> Subs
              </label>
              <label style="display: flex;  gap: 4px;">
                <input
                  checked={type === "c"}
                  on:change={(event) => {
                    type = event.currentTarget.value;
                  }}
                  type="radio"
                  name="type"
                  value="c"
                /> Cheer
              </label>
            </td></tr
          >
          <tr style="height: 40px"
            ><td style="width: 120px;">數量 Count</td><td><input bind:value={count} /></td></tr
          >
        </table>
        {#if !sending}<SolidButton class="sm round" on:click={send}>送出</SolidButton>{/if}
        {#if sending}<div>{sending ? "上傳中" : ""}</div>{/if}
        {#if !sending && success}<div>{success === "s" ? "上傳成功" : "上傳失敗"}</div>{/if}
      </div>
      <div class="control-title">最近50筆斗內</div>
      <div class="card control">
        <table class="control-table">
          {#each list as row}
            <tr>
              <td style="width: 140px; font-size: 12px;"
                >{moment(row.createdAt).format("YYYY-MM-DD HH:mm:ss")}</td
              >
              <td>{row.display}</td>
              <td style="width: 100px;">{row.count} {row.type === "s" ? "Subs" : "Bits"}</td>
              <td style="width: 50px;"
                ><SolidButton class="xs round" on:click={() => deleteItem(row._id)}
                  >刪除</SolidButton
                ></td
              >
            </tr>
          {/each}
        </table>
      </div>
      <div class="control-title">跑馬燈資料</div>
      <div class="card control">
        {#each Object.entries(all) as [key, data]}
          <div>{getText(data)}</div>
        {/each}
      </div>
      <div class="control-title">跑馬燈原始資料</div>
      <SolidButton
        class="sm round"
        on:click={() => {
          allJson = "{}";
        }}>清空</SolidButton
      >
      <div class="card control">
        <textarea bind:value={allJson}></textarea>
      </div>
      {#if jsonStatus}<div style="color:white;">{jsonStatus}</div>{/if}
      <SolidButton class="sm round" on:click={updateJson}>更新</SolidButton>
    </div>
  {:else}
    <div class="card control-status">
      資料庫
      <div class="loading" style="font-size: 12px">{status[dataStatus]}</div>
      TMI
      <div class="loading" style="font-size: 12px">{status[tmiStatus]}</div>
    </div>
  {/if}
</div>

<style lang="scss">
  input {
    background-color: #0e0e10;
    padding: 4px 8px;
    border: none;
    border-radius: 8px;
    font-size: 15px;
    font-weight: 400;
    line-height: 24px;
    width: 100%;
    &::placeholder {
      opacity: 1;
    }
    &:focus {
      outline: none;
      border: none;
    }
  }
  textarea {
    flex: 1 0 4px;
    border: none;
    align-self: stretch;
    resize: none;
    background: transparent;
    box-shadow: none;
    outline: none;
    padding: 0 2px;
    margin: 0;
    border-radius: 7px;
    font-size: 16px;
    line-height: 19px;
    display: block;
    height: 400px;
    overflow-x: hidden !important;
    overflow-y: auto;
    width: 100%;
  }
  .control-bg {
    display: flex;
    justify-content: center;
    align-items: center;
    height: auto;
    min-height: 100%;
    width: 100%;
    padding: 16px;
    background-color: #0e0e10;
  }
  .control-body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 16px;
    width: 100%;
    max-width: 960px;
  }

  :root {
    --white: #ffffff;
    --base: #18181b;
  }

  .card {
    border-radius: 0.2rem;
    border: 1px solid var(--line);
    background: var(--base);
    box-shadow: 0px 10px 40px 0px rgba(0, 0, 0, 0.05);
    font-size: 14px;
    padding: 16px;
    color: #ffffff;
  }
  .control-status {
    width: fit-content;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 4px;
  }

  .control {
    width: 100%;
  }
  .control-title {
    color: #ffffff;
    font-size: 18px;
    font-weight: 700;
    text-align: center;
  }

  .control-table {
    width: 100%;
  }
</style>
