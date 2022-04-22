<script lang="ts">
  import Matter from "matter-js"
  import MatterWrap from "matter-wrap"
  const { Composite, Engine, Runner, Bodies, Mouse, MouseConstraint } = Matter;
  import { onMount } from "svelte";

  Matter.use(MatterWrap);

  let width = 600;
  let height = 600
  let image: HTMLImageElement;
  let files: FileList;

  $: imageWidth = (image && (image.width >= image.height ? 100 : (image.width * 100) / image.height)) || 100
  $: imageHeight = (image && (image.height >= image.width ? 100 : (image.height * 100) / image.width)) || 100

  $: {
    if (files && files[0] !== null) {
      const file = files[0]
      const tempImg = new Image();
      const objectUrl = URL.createObjectURL(file);
      tempImg.src = objectUrl;
      tempImg.addEventListener("load", () => {
        image = tempImg
      })
    } 
  }

  let canvas: HTMLCanvasElement

  onMount(() => {
    image = new Image();
    image.src = "./rock.jpg"
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
      } as any
    });

    const ground = Bodies.rectangle(width / 2, height, width * 2, 60, { isStatic: true });
    const body = Bodies.rectangle(400, 200, imageWidth, imageHeight, {
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

        if (image && body !== ground) {
          context.translate(body.position.x, body.position.y);
          context.rotate(body.angle);
          context.drawImage(image, -imageWidth / 2, -imageHeight / 2, imageWidth, imageHeight);
          context.rotate(-body.angle);
          context.translate(-body.position.x, -body.position.y);
          continue
        }

        for (const part of body.parts) {
          const vertices = part.vertices;
          context.beginPath();
          context.moveTo(vertices[0].x, vertices[0].y)
          for (const vertex of vertices) {
              context.lineTo(vertex.x, vertex.y);
          }
          context.lineTo(vertices[0].x, vertices[0].y);
          context.closePath();
          context.lineWidth = 2
          context.strokeStyle = "rgba(255, 255, 255, 255)";
          context.globalAlpha = part.render.opacity || 1;
          context.fillStyle = part.render.fillStyle || "#ddd";
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

<div class="fixed top-0 left-0 mx-4 my-2">
  <input bind:files={files} accept="image/*" type="file">
</div>

<canvas {width} {height} bind:this={canvas}></canvas>