<script>
  import { onMount } from 'svelte';

  export let activeExperiment = null;

  let canvas;
  let rendererType = 'WebGL';
  let reducedMotion = false;
  let setActiveExperiment = () => {};

  $: setActiveExperiment(activeExperiment);

  onMount(() => {
    let cleanup = () => {};

    const initialize = async () => {
      let THREE;
      let renderer;

      if ('gpu' in navigator) {
        try {
          THREE = await import('three/webgpu');
          renderer = new THREE.default({ canvas, alpha: true, antialias: true });
          await renderer.init();
          rendererType = 'WebGPU';
        } catch {
          // WebGL keeps the directory available on unsupported browsers.
        }
      }

      if (!renderer) {
        THREE = await import('three');
        renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
      }

      let frame;
      let disposed = false;
      let isVisible = !document.hidden;
      const scene = new THREE.Scene();
      scene.fog = new THREE.FogExp2(0x090a0d, 0.075);

      const camera = new THREE.PerspectiveCamera(38, 1, 0.1, 100);
      camera.position.set(0, 0, 12);

      const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
      reducedMotion = prefersReducedMotion.matches;

      const material = new THREE.MeshBasicMaterial({
        color: 0xb8ffa4,
        transparent: true,
        opacity: 0.17,
        side: THREE.DoubleSide,
        blending: THREE.AdditiveBlending
      });
      const edgeMaterial = new THREE.LineBasicMaterial({
        color: 0xd9ffca,
        transparent: true,
        opacity: 0.6
      });
      const inactiveColor = new THREE.Color(0xb8ffa4);
      const activeColor = new THREE.Color(0xffffff);
      const grid = new THREE.Group();
      const cards = [];
      const geometry = new THREE.PlaneGeometry(1.55, 0.98, 10, 10);
      const edgeGeometry = new THREE.EdgesGeometry(geometry);

      for (let row = -3; row <= 3; row += 1) {
        for (let column = -4; column <= 4; column += 1) {
          const experimentIndex =
            row === 0 && column === -3
              ? 0
              : row === 0 && column === -1
                ? 1
                : row === 0 && column === 1
                  ? 2
                  : row === 0 && column === 3
                    ? 3
                    : row === -1 && column === -2
                      ? 4
                      : row === -1 && column === 2
                        ? 5
                        : null;
          const cardMaterial = experimentIndex === null ? material : material.clone();
          const cardEdgeMaterial = experimentIndex === null ? edgeMaterial : edgeMaterial.clone();
          const card = new THREE.Mesh(geometry, cardMaterial);
          const edge = new THREE.LineSegments(edgeGeometry, cardEdgeMaterial);
          const holder = new THREE.Group();
          const baseZ = -Math.abs(column) * 0.28 - Math.abs(row) * 0.12;

          holder.add(card, edge);
          holder.position.set(column * 1.85, row * 1.23, baseZ);
          holder.rotation.set(row * -0.035, column * 0.04, 0);
          holder.userData = {
            baseZ,
            cardMaterial,
            cardEdgeMaterial,
            experimentIndex,
            offset: Math.random() * Math.PI * 2,
            row,
            column
          };
          grid.add(holder);
          cards.push(holder);
        }
      }

      scene.add(grid);

      setActiveExperiment = (index) => {
        activeExperiment = index;
      };

      const targetRotation = new THREE.Vector2();
      const onPointerMove = (event) => {
        targetRotation.set(
          ((event.clientY / window.innerHeight) * 2 - 1) * 0.18,
          ((event.clientX / window.innerWidth) * 2 - 1) * 0.24
        );
      };
      const resize = () => {
        const { width, height } = canvas.getBoundingClientRect();
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
        renderer.setSize(width, height, false);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.75));
      };

      if (disposed) {
        renderer.dispose();
        return;
      }

      resize();
      window.addEventListener('resize', resize);
      window.addEventListener('pointermove', onPointerMove, { passive: true });

      const clock = new THREE.Clock();
      const render = () => {
        if (!isVisible || disposed) return;

        const elapsed = clock.getElapsedTime();
        grid.rotation.x += (targetRotation.x - grid.rotation.x) * 0.025;
        grid.rotation.y += (targetRotation.y - grid.rotation.y) * 0.025;

        cards.forEach((holder) => {
          const { baseZ, cardMaterial, cardEdgeMaterial, column, experimentIndex, offset, row } = holder.userData;
          const isActive = experimentIndex === activeExperiment;
          const isDimmed = activeExperiment !== null && experimentIndex !== null && !isActive;
          const intensity = isActive ? 1 : isDimmed ? 0.16 : 0;

          holder.position.z = baseZ + (reducedMotion ? 0 : Math.sin(elapsed * 0.7 + offset) * 0.13) + intensity * 0.65;
          holder.rotation.z = reducedMotion ? 0 : Math.sin(elapsed * 0.45 + offset) * 0.025;
          holder.scale.lerp(new THREE.Vector3(1 + intensity * 0.18, 1 + intensity * 0.18, 1), 0.1);

          if (experimentIndex !== null) {
            cardMaterial.opacity += ((isActive ? 0.5 : 0.17) - cardMaterial.opacity) * 0.1;
            cardEdgeMaterial.opacity += ((isActive ? 1 : isDimmed ? 0.24 : 0.6) - cardEdgeMaterial.opacity) * 0.1;
            cardMaterial.color.lerp(isActive ? activeColor : inactiveColor, 0.1);
          }
        });

        renderer.render(scene, camera);
        frame = requestAnimationFrame(render);
      };
      const onVisibilityChange = () => {
        isVisible = !document.hidden;
        if (isVisible) {
          clock.getDelta();
          render();
        }
      };

      document.addEventListener('visibilitychange', onVisibilityChange);
      render();

      cleanup = () => {
        disposed = true;
        cancelAnimationFrame(frame);
        window.removeEventListener('resize', resize);
        window.removeEventListener('pointermove', onPointerMove);
        document.removeEventListener('visibilitychange', onVisibilityChange);
        geometry.dispose();
        edgeGeometry.dispose();
        material.dispose();
        edgeMaterial.dispose();
        cards.forEach(({ userData }) => {
          userData.cardMaterial.dispose();
          userData.cardEdgeMaterial.dispose();
        });
        renderer.dispose();
      };
    };

    initialize();

    return () => cleanup();
  });
</script>

<canvas bind:this={canvas} aria-hidden="true"></canvas>
<p class="renderer-label" aria-live="polite">{rendererType} / live field</p>

<style>
  canvas {
    position: fixed;
    inset: 0;
    z-index: 0;
    width: 100%;
    height: 100%;
  }

  .renderer-label {
    position: fixed;
    right: 1.5rem;
    bottom: 1.25rem;
    z-index: 1;
    color: rgba(226, 255, 215, 0.76);
    font: 0.68rem/1 'DM Mono', monospace;
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }
</style>
