<script>
    export let type = 'primary';
    export let flat = false;
    export let inverse = false;
    export let small = false;
    export let end = false;
  </script>
  
  <button class:flat={flat} class:inverse={inverse} class={type} class:end={end} class:small={small} on:click type='button'>
    <slot></slot>
  </button>
  
  <style>
    button{
    width: 100px;
      border: 0;
      cursor: pointer;
      border-radius: 6px;
      padding: 8px 12px;
      font-weight: bold;
      box-shadow: 1px 2px 3px rgba(0,0,0,0.2);
      text-align: center;
    }
    .flat{
      box-shadow: none;
    }
    .primary{
      background: #00789D;
      color: white;
    }
    .secondary{
      background: #45c496;
      color: white;
    }
    .primary.inverse{
      color: #d91b42;
      background: white;
      border: 2px solid #d91b42;
    }
    .secondary.inverse{
      color: #45c496;
      background: white;
      border: 2px solid #45c496;
    }

    .end{
      align-self: end;
    }

    .small{
      width: 130px;
      height: 30px;
    }
  </style>