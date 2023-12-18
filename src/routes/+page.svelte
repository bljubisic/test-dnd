<script>
  // @ts-nocheck
  
    import { flip } from 'svelte/animate';
    import { send, receive } from './transition.js';
    import SurveyNode from "survey-md/SurveyNode.svelte";
    import md from "survey-md/md";

    let questions = [
      {
        name: "Age group",
        question: "How old are you?\n\n? age_group\n- 0-18\n- 19-35\n- 36-55\n- 56 or older",
      },
      {
        name: "Favorite color",
        question: "What are your favorite colors?\n\n?fav_color max=4 min=2\n- Red\n- Blue\n- Green\n- Yellow",
      },
      {
        name: "Open ended question",
        question: "Open Ended Question sample?\n\n?open_ended\nsdjfjdshf",
      },
      {
        name: "Age",
        question: "How old are You?\n\n?age min=1 max=99\nAge",
      }
    ]
    let Stack = [
      {
        name: 'Questions',
        items: questions
      },
      {
        name: 'Survey',
        items: []
      },
    ];
  
    let stackHover;

    async function getNode(question) {
      let node = await md(question);
      return node;
    }
  
    function dragStart(event, stackIndex, itemIndex) {
      const data = { stackIndex, itemIndex };
      event.dataTransfer.dropEffect = 'copy';
      event.dataTransfer.setData('text/plain', JSON.stringify(data));
    }
  
    function drop(event, stackIndex, targetIndex = 0) {
      event.preventDefault();
      const json = event.dataTransfer.getData('text/plain');
      const data = JSON.parse(json);
      console.log(data);
      console.log(Stack);
      // const [item] = Stack[data.stackIndex].items.splice(data.itemIndex, 1);
      if (stackIndex === data.stackIndex) {
        const [item] = Stack[data.stackIndex].items.splice(data.itemIndex, 1);
        Stack[stackIndex].items.push(item);
      } else {
        Stack[stackIndex].items.push(Stack[data.stackIndex].items[data.itemIndex]);
      }
      Stack = Stack;
      stackHover = null;
    }

    const chars =
      "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";

    function uid() {
      let id = "";
      let size = 16;
      while (size--) id += chars[(Math.random() * 62) | 0];
      return id;
    }

    function next() {

    }

    var page = '';

    let context = {
      uid: uid(),
      started_at: new Date()
        .toISOString()
        .substring(0, 19)
        .replace("T", " ")
    };

    var toggle = false;
    let id = "";

    function toggleButton(name) {
      id = name;
      toggle = !toggle;
    }

    function flatten(obj) {
      var flattenedObject = {};
      traverseAndFlatten(obj, flattenedObject);
      return flattenedObject;
    }

    function traverseAndFlatten(currentNode, target, flattenedKey) {
      for (var key in currentNode) {
          if (currentNode.hasOwnProperty(key)) {
              var newKey;
              if (flattenedKey === undefined) {
                  newKey = key;
              } else {
                  newKey = flattenedKey + '.' + key;
              }

              var value = currentNode[key];
              if (typeof value === "object") {
                  traverseAndFlatten(value, target, newKey);
              } else {
                  target[newKey] = value;
              }
          }
      }
    }

    function convert(flatten, raw) {
      var convertedObj = {};
      var keys = Object.keys(flatten);
      var questionKeys = keys.filter((key) => key.includes("question.name"));
      questionKeys.forEach((key) => convertedObj[key.substring(key.lastIndexOf(".")+1)] = flatten[key]);
      convertedObj["type"] = "question"; 
      var questionParamsKeys = keys.filter((key) => key.includes("question.params"));
      if (questionParamsKeys) {
        var params = {}
        questionParamsKeys.forEach((key) => params[key.substring(key.lastIndexOf(".")+1)] = flatten[key]);
        convertedObj["params"] = params;
      }
      var valueKeys = keys.filter((key) => key.includes("value"));
      if (valueKeys) {
        var values = [];
        valueKeys.forEach((key) =>  values.push(flatten[key]));
        if (values.length > 1) {
          convertedObj["values"] = values;
        } else {
          convertedObj["values"] = [];
        }
      }
      convertedObj["raw"] = raw;
      return JSON.stringify(convertedObj);
    }
  </script>
  <p>Drag a framework/library from random to respected drop</p>
  <div class="container">
    <div in:receive={{ key: 0 }} out:send={{ key: 0 }}>
      <b>{Stack[0].name}</b>
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div class="questions"
        class:hovering={stackHover === Stack[0].name}
        on:dragenter={() => (stackHover = Stack[0].name)}
        on:dragleave={() => (stackHover = null)}
        on:drop={(event) => drop(event, 0)}
        ondragover="return false"
      >
        {#each Stack[0].items as item, itemIndex (item)}
          <div
            class="item"
            in:receive={{ key: itemIndex }}
            out:send={{ key: itemIndex }}
            animate:flip={{ duration: 500 }}
          >
            <button class=""on:click={toggleButton(itemIndex)}>✎</button>
            <li draggable={true} on:dragstart={(event) => dragStart(event, 0, itemIndex)}>
              {item.name}
            </li>
          </div>
        {/each}
        </div>
        {#if toggle}
          <textarea bind:value={questions[id].question} rows="10" cols="50"/>
          <button on:click={toggleButton(id)}>Save</button>
        {/if}
    </div>
    <div class="work">
      <b>{Stack[1].name}</b>
      <div in:receive={{ key: 1 }} out:send={{ key: 1 }} class="target-design">
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div class="target"
          class:hovering={stackHover === Stack[1].name}
          on:dragenter={() => (stackHover = Stack[1].name)}
          on:dragleave={() => (stackHover = null)}
          on:drop={(event) => drop(event, 1)}
          ondragover="return false"
        >
          {#each Stack[1].items as item, itemIndex }
            <div
              class="item"
              in:receive={{ key: itemIndex }}
              out:send={{ key: itemIndex }}
            >
              <li draggable={true} on:dragstart={(event) => dragStart(event, 1, itemIndex)}>
                <div class="render">
                  <div class="item">
                    <li>
                      {#await page = md(item.question)}
                        <p>...waiting</p>
                      {:then page}
                        {#each page as node}
                          <SurveyNode {node} bind:context {next} />
                        {/each}
                      {:catch error}
                        <p style="color: red">{error.message}</p>
                      {/await} 
                    </li>
                  </div>
                </div>
              </li>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>

  
  <style>
    .hovering {
      border-color: #454B1B;
    }
    .item {
      display: inline;
      margin-top: 10px; /* required for flip to work */
    }
    li {
      background-color: #00FFFF;
      cursor: pointer;
      display: inline-block;
      margin-right: 10px;
      padding: 10px;
    }
    li:hover {
      background: red;
      color: white;
    }
    p {
      border: solid green 1px;

    }
    button {
      font-size: 20px;
    }
    .work {
      display: flex; /* required for drag & drop to work when .item display is inline */
      flex-direction: column;
      width: 100%; /* needed when empty */
      height: auto;
      padding: 10px;
      border: solid green 1px;
    }
    .questions {
      display: flex; /* required for drag & drop to work when .item display is inline */
      flex-direction: column;
      flex-grow: 1;
      align-items: stretch;
      width: 80px; /* needed when empty */
      height: auto;
      padding: 10px;
      border: solid green 1px;
    }
    .target-design {
      border: solid green 1px;
    }
    .target {
      display: flex; /* required for drag & drop to work when .item display is inline */
      flex-direction: column;
      width: 95%; /* needed when empty */
      height: 100%;
      padding: 10px;
      margin-left: auto;
      margin-right: auto;
    }
    .container {
      display: flex;
      flex-direction: row;
    }
    .results {
      display: flex;
      flex-direction: column;
      margin-top: 20px;
    }
    .markdown {
      display: flex;
      flex-direction: column;
    }
    .render {
      display: flex;
      flex-direction: column;
    }
    .container-results {
      display: flex;
      flex-direction: row;
      margin-top: 20px;
    }
  </style>
  