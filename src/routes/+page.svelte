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
      event.dataTransfer.setData('text/plain', JSON.stringify(data));
    }
  
    function drop(event, stackIndex) {
      event.preventDefault();
      const json = event.dataTransfer.getData('text/plain');
      const data = JSON.parse(json);
      const [item] = Stack[data.stackIndex].items.splice(data.itemIndex, 1);
      Stack[stackIndex].items.push(item);
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
  </script>
  
  <p>Drag a framework/library from random to respected drop</p>
  <div in:receive={{ key: 0 }} out:send={{ key: 0 }}>
    <b>{Stack[0].name}</b>
    <p
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
          <li draggable={true} on:dragstart={(event) => dragStart(event, 0, itemIndex)}>
            {item.name}
          </li>
        </div>
      {/each}
    </p>
  </div>
  <div in:receive={{ key: 1 }} out:send={{ key: 1 }}>
    <b>{Stack[1].name}</b>
    <p
      class:hovering={stackHover === Stack[1].name}
      on:dragenter={() => (stackHover = Stack[1].name)}
      on:dragleave={() => (stackHover = null)}
      on:drop={(event) => drop(event, 1)}
      ondragover="return false"
    >
      {#each Stack[1].items as item, itemIndex (item)}
        <div
          class="item"
          in:receive={{ key: itemIndex }}
          out:send={{ key: itemIndex }}
          animate:flip={{ duration: 500 }}
        >
          <li draggable={true} on:dragstart={(event) => dragStart(event, 1, itemIndex)}>
            {item.name}
          </li>
        </div>
      {/each}
    </p>
  </div>
  <div>
    {#each Stack[1].items as item, itemIndex (item)}
      <div
        class="item"
      >
        <li>
          <span style="white-space: pre-line">{item.question}</span>
        </li>
      </div>
    {/each}
  </div>
  <div>
    {#each Stack[1].items as item, itemIndex (item)}
      <div
        class="item"
      >
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
    {/each}
  </div>
  
  <!-- {#each Stack as stack, stackIndex (stack)}
    <div animate:flip in:receive={{ key: stackIndex }} out:send={{ key: stackIndex }}>
      <b>{stack.name}</b>
      <p
        class:hovering={stackHover === stack.name}
        on:dragenter={() => (stackHover = stack.name)}
        on:dragleave={() => (stackHover = null)}
        on:drop={(event) => drop(event, stackIndex)}
        ondragover="return false"
      >
        {#each stack.items as item, itemIndex (item)}
          <div
            class="item"
            in:receive={{ key: itemIndex }}
            out:send={{ key: itemIndex }}
            animate:flip={{ duration: 500 }}
          >
            <li draggable={true} on:dragstart={(event) => dragStart(event, stackIndex, itemIndex)}>
              {item}
            </li>
          </div>
        {/each}
      </p>
    </div>
  {/each} -->
  
  <style>
    .hovering {
      border-color: #454B1B;
    }
    .item {
      display: inline; /* required for flip to work */
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
      display: flex; /* required for drag & drop to work when .item display is inline */
      height: 40px; /* needed when empty */
      padding: 10px;
    }
  </style>
  