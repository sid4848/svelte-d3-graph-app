<script>
  import Sankey from "./SankeyNivo.svelte";
  import graph from "./graph";

  let width = 600;
  let height = 400;

</script>

<style>
  
  div {  
    align-items: center;
    padding: 1rem;
    border-radius: 6px;
    box-shadow: 0.1px 2px 4px 1px rgba(0,0,0,0.5);
    width: 50vw;
    margin:auto auto;
    margin-top: 5rem;
  }
</style>

<!-- <svelte:window bind:innerWidth={width} /> -->
<div >
  <Sankey
    {graph}
    {width}
    {height}
    extent={[[1, 1], [width - 1, height - 6]]} />
</div>