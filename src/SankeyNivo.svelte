<script>
    import {
      sankey as d3sankey,
      // SankeyGraph,
      // SankeyLink,
      // SankeyNode,
      sankeyLeft,
      sankeyRight,
      sankeyCenter,
      sankeyJustify,
    } from "d3-sankey";
    import { linkHorizontal } from "d3-shape";
    import { scaleSequential } from "d3-scale";
    import { interpolateCool } from "d3-scale-chromatic";
    import { extent as d3Extent } from "d3-array";
  
    import Group from "./Group.svelte";
  
    export let width;
    export let height;
    export let margin = {
      top: 0,
      left: 0,
      right: 200,
      bottom: 0,
    };
  
    export let graph;
    export let extent = undefined;
    
  
    const color = scaleSequential(interpolateCool);
  
    let nodes, links;
    let nodesOpacity 
    let nodesStrokeColor 

    $: {

       nodesOpacity = 0.7
       nodesStrokeColor = "white"
      
      const sankey = d3sankey();
      if (extent) sankey.extent(extent);
  
      const data = sankey(graph);
      nodes = data.nodes;
      links = data.links;
      console.log(links)

  
      // Set color domain after sankey() has set depth
      color.domain(d3Extent(nodes, (d) => d.depth));
    }


  
    const path = linkHorizontal()
      // @ts-ignore
      .source((d) => [d.source.x1, d.y0])
      // @ts-ignore
      .target((d) => [d.target.x0, d.y1]);

    let highlightLinkIndexes = [];
  </script>
  
  <svg
    width={width + margin.left + margin.right}
    height={height + margin.top + margin.bottom}>
    <g>
      <g>
        {#each links as link, i (`link-${i}`)}
          <!-- svelte-ignore a11y-mouse-events-have-key-events -->
          <path
          key={`link-${i}`}
          d={path(link) || undefined}
          stroke={(link.index % 2 === 0) ? 
          highlightLinkIndexes.includes(i) ? "blue" : "blue" :
          highlightLinkIndexes.includes(i) ? "red" : "red"
          }

          stroke-width={Math.max(1, link.width)}
          opacity={highlightLinkIndexes.includes(i) ? 0.5 : 0.15}
          fill="none"
          on:mouseover={(e) => {
            highlightLinkIndexes = [i];
            const label = document.getElementById(`label-${i}`);
            if (label) {
              label.style.opacity = 1;
            }
          }}
          on:mouseout={(e) => {
            highlightLinkIndexes = [];
            const label = document.getElementById(`label-${i}`);
            if (label) {
              label.style.opacity = 0;
            }
          }} />

        <!-- Add label for the link value (date and time) -->
        <text
          id={`label-${i}`}
          x={(link.source.x1 + link.target.x0) / 2}
          y={(link.y0 + link.y1) / 2}
          dy="0.35em"
          style="font-size: 10px; fill: black; text-anchor: middle; opacity: 0; font-weight:600">
          {new Date(link.value).toLocaleString()} <!-- Format date and time -->
        </text>
        {/each}
      </g>
  
      {#each nodes as node, i (`node-${i}`)}
        <Group top={node.y0} left={node.x0}>
          <!-- svelte-ignore a11y-mouse-events-have-key-events -->
          <rect
            id={`rect-${i}`}
            width={node.x1 - node.x0}
            height={node.y1 - node.y0}
            fill={color(node.depth)}
            opacity={nodesOpacity}
            stroke={nodesStrokeColor}
            stroke-width={2}
            on:mouseover={(e) => {
              highlightLinkIndexes = [...node.sourceLinks.map((l) => l.index), ...node.targetLinks.map((l) => l.index)];
              const n = document.getElementById(`rect-${i}`);
              if (n) {
                nodesOpacity = 1;
                nodesStrokeColor = "purple"
              }
            }}
            on:mouseout={(e) => {
              highlightLinkIndexes = [];
              const n = document.getElementById(`rect-${i}`);
              if (n) {
                nodesOpacity = 0.7;
                nodesStrokeColor = "white"
              }
            }} />
  
          <text
            x={30}
            y={(node.y1 - node.y0) / 2}
            dy={5}
            style="font: 5px, font-family:sans-serif,font-weight:600"
            _verticalAnchor="middle">
            {node.name}
          </text>
        </Group>
      {/each}
  
    </g>
  </svg>