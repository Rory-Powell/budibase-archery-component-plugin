<script>
  import { getContext } from "svelte"
  const { styleable } = getContext("sdk")
  const component = getContext("component")

    export let targetType
    export let arrow
    const colorMapping = {
      olympic: {
        0: 'miss',
        1: 'white',
        2: 'white',
        3: 'black',
        4: 'black',
        5: 'blue',
        6: 'blue',
        7: 'red',
        8: 'red',
        9: 'yellow',
        10: 'yellow'
      },
      nfaa: {
        0: 'miss',
        1: 'nfaa-blue',
        2: 'nfaa-blue',
        3: 'nfaa-blue',
        4: 'nfaa-blue',
        5: 'white'
      }
    }

</script>

<div class={`score ${colorMapping[targetType][arrow.score]}`}>
    {arrow.score}
</div>


<style>
    .score {
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 50%;
        width: 35px;
        height: 35px;
        margin: 3px;
        /*padding-top: 5px;*/
        box-shadow: 0 5px 11px 0 rgba(0, 0, 0, 0.18), 0 4px 15px 0 rgba(0, 0, 0, 0.15);
    }

    .red {
        color: white;
        background-color: #dc3545;
    }

    .blue {
        color: white;
        background-color: #17a2b8;
    }

    .yellow {
        color: #444;
        background-color: #ffc107;
        font-weight: 400;
    }

    .white {
        color: #444;
        background-color: white;
        font-weight: 400;
    }

    .black {
        color: white;
        background-color: black;
    }

    .miss {
        color: white;
        background-color: #444;
    }

    .nfaa-blue {
        color: white;
        background-color: #233b5e;
    }

</style>