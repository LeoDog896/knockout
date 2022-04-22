<script lang="ts">
  import Matter from "matter-js"
  import MatterWrap from "matter-wrap"
  const { Composite, Engine, Runner, Bodies, Mouse, MouseConstraint } = Matter;
  import { onMount } from "svelte";

  Matter.use(MatterWrap);

  let width = 600;
  let height = 600
  let canvas: HTMLCanvasElement

  onMount(() => {
    width = window.innerWidth
    height = window.innerHeight

    const engine = Engine.create()
    const context = canvas.getContext("2d")
    if (context == null) return;

    // add mouse control
    const mouse = Mouse.create(canvas);
    const mouseConstraint = MouseConstraint.create(engine, {
      mouse: mouse,
      constraint: {
        stiffness: 0.2,
        render: {
          visible: false
        }
      }
    });

    const ground = Bodies.rectangle(width / 2, height, width, 60, { isStatic: true });
    const body = Bodies.rectangle(400, 200, 80, 80, {
      plugin: {
        wrap: {
          min: {
            x: 0,
            y: 0
          },
          max: {
            x: width,
            y: height
          }
        }
      }
    })

    Composite.add(engine.world, [body, mouseConstraint, ground])

    const runner = Runner.create();

    Runner.run(runner, engine);
    (function render() {
      const bodies = Composite.allBodies(engine.world);
      window.requestAnimationFrame(render);
      context.fillStyle = '#ffffff';
      context.fillRect(0, 0, canvas.width, canvas.height);
      for (const body of bodies) {
        for (const part of body.parts) {
          const vertices = part.vertices;
          context.beginPath();
          context.moveTo(vertices[0].x, vertices[0].y)
          for (const vertex of vertices) {
              context.lineTo(vertex.x, vertex.y);
          }
          context.lineTo(vertices[0].x, vertices[0].y);
          context.closePath();
          context.lineWidth = 0
          context.strokeStyle = "rgba(1, 1, 1, 0)";
          context.globalAlpha = part.render.opacity;
          context.fillStyle = part.render.fillStyle;
          context.fill();
        }
      }
    })();
  })
</script>

<svelte:window on:resize={() => {
  width = window.innerWidth
  height = window.innerHeight
}}></svelte:window>

<canvas {width} {height} bind:this={canvas}></canvas>