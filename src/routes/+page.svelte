<script>
  // @ts-nocheck
  
    import { flip } from 'svelte/animate';
    import { send, receive } from './transition.js';
    let RandomStack = [
      {
        name: 'Random',
        items: ['React', 'Angular', 'Solid', 'Rails', 'Express', 'Flask', 'VPS']
      },
      {
        name: 'Frontend Stack',
        items: ['Svelte', 'SvelteKit']
      },
      {
        name: 'Backend Stack',
        items: ['Django']
      },
      {
        name: 'Server Stack',
        items: ['Vercel']
      }
    ];
  
    let stackHover;
  
    function dragStart(event, stackIndex, itemIndex) {
      const data = { stackIndex, itemIndex };
      event.dataTransfer.setData('text/plain', JSON.stringify(data));
    }
  
    function drop(event, stackIndex) {
      event.preventDefault();
      const json = event.dataTransfer.getData('text/plain');
      const data = JSON.parse(json);
      const [item] = RandomStack[data.stackIndex].items.splice(data.itemIndex, 1);
      RandomStack[stackIndex].items.push(item);
      RandomStack = RandomStack;
  
      stackHover = null;
    }
  </script>
  
  <p>Drag a framework/library from random to respected drop</p>
  
  {#each RandomStack as stack, stackIndex (stack)}
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
  {/each}
  
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
  