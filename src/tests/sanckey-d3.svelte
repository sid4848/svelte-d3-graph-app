<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
  
	// Sample data representing users and chat relationships
	let users = [
	  {
		id: 1,
		name: 'A',
		contacts: [
		  { id: 2, date: '2023-07-22', time: '10:00:00' },
		  { id: 3, date: '2023-07-21', time: '12:30:00' },
		  { id: 4, date: '2023-07-20', time: '16:45:00' }
		]
	  },
	  {
		id: 2,
		name: 'B',
		contacts: [
		  { id: 1, date: '2023-07-22', time: '10:00:00' },
		  { id: 3, date: '2023-07-21', time: '11:15:00' },
		  { id: 5, date: '2023-07-19', time: '09:30:00' }
		]
	  },
	  {
		id: 3,
		name: 'C',
		contacts: [
		  { id: 1, date: '2023-07-21', time: '12:30:00' },
		  { id: 2, date: '2023-07-21', time: '11:15:00' },
		  { id: 5, date: '2023-07-20', time: '14:00:00' }
		]
	  },
	  {
		id: 4,
		name: 'D',
		contacts: [
		  { id: 1, date: '2023-07-20', time: '16:45:00' },
		  { id: 5, date: '2023-07-20', time: '15:30:00' },
		  { id: 2, date: '2023-07-18', time: '08:45:00' }
		]
	  },
	  {
		id: 5,
		name: 'E',
		contacts: [
		  { id: 2, date: '2023-07-19', time: '09:30:00' },
		  { id: 3, date: '2023-07-20', time: '14:00:00' },
		  { id: 4, date: '2023-07-20', time: '15:30:00' }
		]
	  }
	];
  
	// Function to create an array of edges based on the chat relationships
	function createEdges(users) {
	  const edges = [];
	  users.forEach(user => {
		user.contacts.forEach(contact => {
		  edges.push({
			source: user.id,
			target: contact.id,
			date: contact.date,
			time: contact.time
		  });
		});
	  });
	  return edges;
	}
  
	// Call the function to create the edges based on the sample user data
	let links = createEdges(users);


  
	onMount(() => {
    // Create a D3 force simulation to layout the graph
    const simulation = d3.forceSimulation(users)
	  .force('link', d3.forceLink(links).id(d => d.id))
      .force('charge', d3.forceManyBody().strength(-1800)) // Increase the strength value to increase node repulsion
      .force('center', d3.forceCenter(250, 150))
	  .on('tick', ticked);
	  

    // Create SVG elements for links and nodes
    const svg = d3.select('svg');

    // Append the lines (edges) to the SVG
	const link = svg.selectAll('line')
		.data(links)
		.enter().append('line')
		.attr('stroke', '#999')
		.attr('stroke-opacity', '0.6')
		.attr('stroke-width', '1.5'); // Increase the line thickness from 1 to 3 or any desired value


	// Append date and time labels for chat relationships to the lines
    // const linkLabels = svg.selectAll('.link-label')
    //   .data(links)
    //   .enter().append('text')
    //   .attr('class', 'link-label')
    //   .attr('text-anchor', 'middle')
    //   .attr('dy', '-0.3em')
    //   .attr('font-size', '10px')
    //   .attr('fill', '#999')
    //   .text(d => `${d.date} ${d.time}`);

    // Append the circles (nodes) to the SVG
	const node = svg.selectAll('circle')
		.data(users)
		.enter().append('circle')
		.attr('r', 30) // Increase the circle size from 10 to 20
		.attr('fill', 'rgba(245, 0, 172)')
		.attr('stroke', 'white')
		.attr('stroke-width', '0')
		.attr('transition', '1s')
		.on('mouseover', function (event, d) {
        d3.select(this)
          .transition(2)
          .duration(200)
          .attr('r', 35)
          .attr('stroke-width', '12')
		  .attr('stroke', 'rgba(52, 235, 204, 0.5)'); // Add stroke color change during hover
      })
      .on('mouseout', function (event, d) {
        d3.select(this)
          .transition(2)
          .duration(200)
          .attr('r', 30)
          .attr('stroke-width', '0')
		  .attr('stroke', 'white'); // Add stroke color change during hover
      });

    // Append text labels for user names to the circles
    const text = svg.selectAll('text')
      .data(users)
      .enter().append('text')
      .attr('text-anchor', 'middle')
      .attr('dy', '0.35em')
      .attr('font-size', '12px') // Increase font size for text
      .attr('fill', 'white')
      .text(d => d.name);

	  function ticked() {
      // Update positions of nodes, text labels, and links on each tick of the simulation
      link
        .attr('x1', d => d.source.x)
        .attr('y1', d => d.source.y)
        .attr('x2', d => d.target.x)
        .attr('y2', d => d.target.y);

      node
        .attr('cx', d => d.x)
        .attr('cy', d => d.y);

      // Update text label positions based on the nodes
      text
        .attr('x', d => d.x)
        .attr('y', d => d.y);

	// Update link label positions based on the midpoints between the source and target nodes
	// linkLabels
    //     .attr('x', d => (d.source.x + d.target.x) / 2)
    //     .attr('y', d => (d.source.y + d.target.y) / 2);
    
    }

    // Function to apply the left-right moving animation to circles
    function applyBubbleAnimation() {
		node
        .transition()
        .duration(2000)
        .attr('cx', d => d.x + 5) // Move right by 5 units
        .transition()
        .duration(2000)
        .attr('cx', d => d.x - 5) // Move left by 5 units
        .on('end', applyBubbleAnimation); // Reapply animation on completion

      text
        .transition()
        .duration(2000)
        .attr('x', d => d.x + 5) // Move right by 5 units
        .transition()
        .duration(2000)
        .attr('x', d => d.x - 5) // Move left by 5 units
        .on('end', applyBubbleAnimation); // Reapply animation on completion
    }
    

    // Start the bubble animation
    applyBubbleAnimation();

	

    // Update positions of nodes, text labels, and links on each tick of the simulation
    // simulation.on('tick', () => {
    //   link
    //     .attr('x1', d => d.source.x)
    //     .attr('y1', d => d.source.y)
    //     .attr('x2', d => d.target.x)
    //     .attr('y2', d => d.target.y);

    //   node
    //     .attr('cx', d => d.x)
    //     .attr('cy', d => d.y);

    //   // Update text label positions based on the nodes
    //   text
    //     .attr('x', d => d.x)
    //     .attr('y', d => d.y);
    // });
  });
  </script>
  
  <style>
	

  circle {
    transition: r 0.2s, fill 0.2s, stroke-width 0.2s, stroke 0.2s;
    animation: bubblesAnimation 3s infinite;
  }

  @keyframes bubblesAnimation {
    0%, 100% {
      transform: scale(1); /* Normal scale */
    }
    50% {
      transform: scale(1.1); /* Scale up */
    }
  }
  
	svg {
		width: 100vw;
		height: 100vh;
		color: white;
		font-size: 1.2rem;
		font-weight: 600;
	}
	div {
		display: flex;
		align-items: center;
		justify-content: center;
	}
  </style>
  
  <div>
	<svg ></svg>
  </div>
  