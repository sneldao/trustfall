<script>
  import { createEventDispatcher, onMount } from 'svelte';

  export let projects = [];

  const dispatch = createEventDispatcher();
  let canvas;
  let rendererType = 'WebGL';

  onMount(() => {
    let cleanup = () => {};

    const initialize = async () => {
      let THREE;
      let TSL;
      let renderer;
      let isWebGPU = false;

      if ('gpu' in navigator) {
        try {
          THREE = await import('three/webgpu');
          TSL = await import('three/tsl');
          renderer = new THREE.default({ canvas, alpha: false, antialias: true });
          await renderer.init();
          rendererType = 'WebGPU';
          isWebGPU = true;
        } catch {
          // The WebGL treatment preserves navigation on unsupported devices.
        }
      }

      if (!renderer) {
        THREE = await import('three');
        renderer = new THREE.WebGLRenderer({ canvas, alpha: false, antialias: true });
      }

      const scene = new THREE.Scene();
      scene.background = new THREE.Color(0x03030b);
      const camera = new THREE.PerspectiveCamera(42, 1, 0.1, 100);
      camera.position.z = 12;

      const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
      const textureCache = projects.map((project, index) => createProjectTexture(THREE, project, index));
      const cardGeometry = createRoundedPlane(THREE, 3.8, 2.45, 0.26);
      const glassGeometry = createRoundedPlane(THREE, 3.92, 2.57, 0.31);
      const cards = [];
      const columns = 7;
      const rows = 5;
      const spacingX = 4.22;
      const spacingY = 2.86;
      const worldWidth = columns * spacingX;
      const worldHeight = rows * spacingY;
      const scroll = new THREE.Vector2(0, 0);
      const target = new THREE.Vector2(0, 0);
      const velocity = new THREE.Vector2(0, 0);
      const raycaster = new THREE.Raycaster();
      const pointer = new THREE.Vector2();
      const pointerDown = new THREE.Vector2();
      let draggedDistance = 0;
      let dragging = false;
      let hoveredCard = null;
      let frame;
      let disposed = false;
      let visible = !document.hidden;

      for (let row = 0; row < rows; row += 1) {
        for (let column = 0; column < columns; column += 1) {
          const poolIndex = row * columns + column;
          const projectIndex = poolIndex % projects.length;
          const project = projects[projectIndex];
          const imageMaterial = createLiquidMaterial(
            THREE,
            TSL,
            textureCache[projectIndex],
            isWebGPU
          );
          const image = new THREE.Mesh(cardGeometry, imageMaterial);
          const glassMaterial = new THREE.MeshPhysicalMaterial({
            color: project.palette[0],
            transparent: true,
            opacity: isWebGPU ? 0.32 : 0.2,
            roughness: 0.12,
            metalness: 0,
            transmission: isWebGPU ? 0.35 : 0,
            thickness: 0.55,
            ior: 1.42,
            iridescence: 0.32,
            iridescenceIOR: 1.28,
            clearcoat: 1,
            clearcoatRoughness: 0.08,
            side: THREE.DoubleSide,
            depthWrite: false
          });
          const glass = new THREE.Mesh(glassGeometry, glassMaterial);
          glass.position.z = 0.055;

          const holder = new THREE.Group();
          holder.add(image, glass);
          holder.userData = {
            project,
            projectIndex,
            column,
            row,
            imageMaterial,
            glassMaterial,
            baseScale: 1,
            phase: poolIndex * 0.47
          };
          scene.add(holder);
          cards.push(holder);
        }
      }

      const ambient = new THREE.HemisphereLight(0xe7f5ff, 0x190b2d, 2.1);
      scene.add(ambient);
      const key = new THREE.DirectionalLight(0xffffff, 3.2);
      key.position.set(-4, 5, 7);
      scene.add(key);
      const rim = new THREE.DirectionalLight(0x7dc9ff, 2.6);
      rim.position.set(7, -3, 4);
      scene.add(rim);

      const wrap = (value, size) => ((((value + size / 2) % size) + size) % size) - size / 2;

      const layoutCards = (elapsed) => {
        cards.forEach((card) => {
          const { column, row, phase } = card.userData;
          const x = wrap((column - (columns - 1) / 2) * spacingX + scroll.x, worldWidth);
          const y = wrap(((rows - 1) / 2 - row) * spacingY + scroll.y, worldHeight);
          const nx = x / (worldWidth * 0.5);
          const ny = y / (worldHeight * 0.5);
          const radial = nx * nx + ny * ny;
          const bendZ = -radial * 5.4;
          const jelly = reducedMotion ? 0 : Math.sin(elapsed * 1.25 + phase) * 0.035 + velocity.length() * 0.003;

          card.position.set(x, y, bendZ);
          card.rotation.x = ny * -0.46 + jelly;
          card.rotation.y = nx * 0.56 - jelly;
          card.rotation.z = reducedMotion ? 0 : velocity.x * -0.0008 + Math.sin(elapsed * 0.5 + phase) * 0.008;

          const edgeScale = Math.max(0.68, 1 - radial * 0.17);
          const hoverScale = card === hoveredCard ? 1.055 : 1;
          const velocityStretch = Math.min(0.045, velocity.length() * 0.0007);
          card.scale.set(edgeScale * hoverScale + velocityStretch, edgeScale * hoverScale - velocityStretch, 1);
          const interaction = card === hoveredCard ? 1 : 0;
          card.userData.glassMaterial.opacity +=
            ((interaction ? 0.48 : isWebGPU ? 0.32 : 0.2) -
              card.userData.glassMaterial.opacity) *
            0.12;
          updateLiquidMaterial(
            card.userData.imageMaterial,
            elapsed,
            Math.min(1, velocity.length() * 0.02),
            interaction
          );
        });
      };

      const updatePointer = (event) => {
        pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
        pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
      };

      const pickCard = (event) => {
        updatePointer(event);
        raycaster.setFromCamera(pointer, camera);
        const intersections = raycaster.intersectObjects(cards, true);
        return intersections.length ? intersections[0].object.parent : null;
      };

      const setHoveredCard = (card) => {
        if (hoveredCard === card) return;
        hoveredCard = card;
        canvas.style.cursor = dragging ? 'grabbing' : card ? 'pointer' : 'grab';
        if (card) dispatch('projectfocus', card.userData.project);
        else dispatch('projectblur');
      };

      const onPointerDown = (event) => {
        dragging = true;
        draggedDistance = 0;
        pointerDown.set(event.clientX, event.clientY);
        velocity.set(0, 0);
        canvas.setPointerCapture(event.pointerId);
        canvas.style.cursor = 'grabbing';
      };

      const onPointerMove = (event) => {
        if (dragging) {
          const movementX = event.clientX - pointerDown.x;
          const movementY = event.clientY - pointerDown.y;
          draggedDistance += Math.abs(movementX) + Math.abs(movementY);
          target.x += movementX * 0.0085;
          target.y -= movementY * 0.0085;
          velocity.set(movementX, -movementY);
          pointerDown.set(event.clientX, event.clientY);
          setHoveredCard(null);
          return;
        }

        setHoveredCard(pickCard(event));
      };

      const onPointerUp = (event) => {
        if (!dragging) return;
        dragging = false;
        canvas.releasePointerCapture(event.pointerId);
        const card = pickCard(event);
        setHoveredCard(card);

        if (draggedDistance < 12 && card?.userData.project.href) {
          window.location.href = card.userData.project.href;
        }
      };

      const resize = () => {
        const width = window.innerWidth;
        const height = window.innerHeight;
        camera.aspect = width / height;
        camera.fov = width < 640 ? 48 : 42;
        camera.position.z = width < 640 ? 11.8 : 12;
        camera.updateProjectionMatrix();
        renderer.setSize(width, height, false);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, width < 640 ? 1.35 : 1.75));
      };

      resize();
      window.addEventListener('resize', resize);
      canvas.addEventListener('pointerdown', onPointerDown);
      canvas.addEventListener('pointermove', onPointerMove);
      canvas.addEventListener('pointerup', onPointerUp);
      const onPointerLeave = () => !dragging && setHoveredCard(null);

      canvas.addEventListener('pointercancel', onPointerUp);
      canvas.addEventListener('pointerleave', onPointerLeave);

      const timer = new THREE.Timer();
      timer.connect(document);
      const render = (timestamp) => {
        if (!visible || disposed) return;

        timer.update(timestamp);
        const elapsed = timer.getElapsed();
        if (!dragging) {
          target.addScaledVector(velocity, 0.006);
          velocity.multiplyScalar(0.925);
        }
        scroll.lerp(target, 0.16);
        layoutCards(elapsed);
        renderer.render(scene, camera);
        frame = requestAnimationFrame(render);
      };

      const onVisibilityChange = () => {
        visible = !document.hidden;
        if (visible) render();
      };

      document.addEventListener('visibilitychange', onVisibilityChange);
      render();

      cleanup = () => {
        disposed = true;
        cancelAnimationFrame(frame);
        window.removeEventListener('resize', resize);
        canvas.removeEventListener('pointerdown', onPointerDown);
        canvas.removeEventListener('pointermove', onPointerMove);
        canvas.removeEventListener('pointerup', onPointerUp);
        canvas.removeEventListener('pointercancel', onPointerUp);
        canvas.removeEventListener('pointerleave', onPointerLeave);
        document.removeEventListener('visibilitychange', onVisibilityChange);
        timer.dispose();
        cardGeometry.dispose();
        glassGeometry.dispose();
        textureCache.forEach((texture) => texture.dispose());
        cards.forEach((card) => {
          card.userData.imageMaterial.dispose();
          card.userData.glassMaterial.dispose();
        });
        renderer.dispose();
      };
    };

    initialize();
    return () => cleanup();
  });

  function createLiquidMaterial(THREE, TSL, map, isWebGPU) {
    if (isWebGPU) {
      const {
        abs,
        float,
        length,
        max,
        min,
        mix,
        smoothstep,
        texture,
        uniform,
        uv,
        vec2,
        vec3,
        vec4
      } = TSL;
      const material = new THREE.MeshBasicNodeMaterial({
        transparent: true,
        depthWrite: true
      });
      const time = uniform(0);
      const motion = uniform(0);
      const focus = uniform(0);
      const localUv = uv();
      const centered = localUv.sub(0.5);
      const q = abs(centered).sub(vec2(0.43, 0.39));
      const distance = length(max(q, vec2(0)))
        .add(min(max(q.x, q.y), float(0)))
        .sub(0.085);
      const mask = smoothstep(0.018, -0.004, distance);
      const edge = smoothstep(-0.16, -0.01, distance);
      const wave = centered.y
        .mul(14)
        .add(time.mul(1.7))
        .sin()
        .mul(motion.mul(0.014));
      const refractOffset = centered
        .mul(float(1).sub(edge).mul(0.075).add(focus.mul(0.008)))
        .add(vec2(wave, wave.mul(-0.45)));
      const red = texture(map, localUv.add(refractOffset.mul(1.18))).r;
      const green = texture(map, localUv.add(refractOffset)).g;
      const blue = texture(map, localUv.add(refractOffset.mul(0.82))).b;
      const refracted = vec3(red, green, blue);
      const rim = float(1).sub(edge).mul(0.9);
      const highlight = smoothstep(0.18, 0.92, localUv.y)
        .mul(rim)
        .mul(vec3(0.72, 0.9, 1));
      material.colorNode = vec4(
        mix(refracted, refracted.add(highlight), rim.mul(0.72)),
        mask
      );
      material.opacityNode = mask;
      material.userData.liquidUniforms = { focus, motion, time };
      return material;
    }

    const material = new THREE.ShaderMaterial({
      transparent: true,
      depthWrite: true,
      uniforms: {
        map: { value: map },
        uFocus: { value: 0 },
        uMotion: { value: 0 },
        uTime: { value: 0 }
      },
      vertexShader: `
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `,
      fragmentShader: `
        uniform sampler2D map;
        uniform float uFocus;
        uniform float uMotion;
        uniform float uTime;
        varying vec2 vUv;

        float roundedBox(vec2 point, vec2 halfSize, float radius) {
          vec2 q = abs(point) - halfSize;
          return length(max(q, 0.0)) + min(max(q.x, q.y), 0.0) - radius;
        }

        void main() {
          vec2 centered = vUv - 0.5;
          float distance = roundedBox(centered, vec2(0.43, 0.39), 0.085);
          float mask = smoothstep(0.018, -0.004, distance);
          float edge = smoothstep(-0.16, -0.01, distance);
          float wave = sin(centered.y * 14.0 + uTime * 1.7) * uMotion * 0.014;
          vec2 offset = centered * ((1.0 - edge) * 0.075 + uFocus * 0.008);
          offset += vec2(wave, wave * -0.45);
          vec3 refracted = vec3(
            texture2D(map, vUv + offset * 1.18).r,
            texture2D(map, vUv + offset).g,
            texture2D(map, vUv + offset * 0.82).b
          );
          float rim = (1.0 - edge) * 0.9;
          vec3 highlight = smoothstep(0.18, 0.92, vUv.y) * rim * vec3(0.72, 0.9, 1.0);
          gl_FragColor = vec4(mix(refracted, refracted + highlight, rim * 0.72), mask);
        }
      `
    });
    material.userData.liquidUniforms = material.uniforms;
    return material;
  }

  function updateLiquidMaterial(material, time, motion, focus) {
    const uniforms = material.userData.liquidUniforms;
    if (!uniforms) return;

    if (material.isShaderMaterial) {
      uniforms.uTime.value = time;
      uniforms.uMotion.value += (motion - uniforms.uMotion.value) * 0.12;
      uniforms.uFocus.value += (focus - uniforms.uFocus.value) * 0.12;
      return;
    }

    uniforms.time.value = time;
    uniforms.motion.value += (motion - uniforms.motion.value) * 0.12;
    uniforms.focus.value += (focus - uniforms.focus.value) * 0.12;
  }

  function createRoundedPlane(THREE, width, height, radius) {
    const x = -width / 2;
    const y = -height / 2;
    const shape = new THREE.Shape();
    shape.moveTo(x + radius, y);
    shape.lineTo(x + width - radius, y);
    shape.quadraticCurveTo(x + width, y, x + width, y + radius);
    shape.lineTo(x + width, y + height - radius);
    shape.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
    shape.lineTo(x + radius, y + height);
    shape.quadraticCurveTo(x, y + height, x, y + height - radius);
    shape.lineTo(x, y + radius);
    shape.quadraticCurveTo(x, y, x + radius, y);
    const geometry = new THREE.ShapeGeometry(shape, 8);
    geometry.computeBoundingBox();
    const box = geometry.boundingBox;
    const size = new THREE.Vector2();
    box.getSize(size);
    const uv = geometry.attributes.uv;
    const position = geometry.attributes.position;
    for (let index = 0; index < uv.count; index += 1) {
      uv.setXY(index, (position.getX(index) - box.min.x) / size.x, (position.getY(index) - box.min.y) / size.y);
    }
    return geometry;
  }

  function createProjectTexture(THREE, project, index) {
    const width = 1024;
    const height = 660;
    const art = document.createElement('canvas');
    art.width = width;
    art.height = height;
    const context = art.getContext('2d');
    const gradient = context.createLinearGradient(0, 0, width, height);
    gradient.addColorStop(0, project.palette[0]);
    gradient.addColorStop(0.5, project.palette[1]);
    gradient.addColorStop(1, project.palette[2]);
    context.fillStyle = gradient;
    context.fillRect(0, 0, width, height);

    context.globalCompositeOperation = 'screen';
    for (let ring = 0; ring < 18; ring += 1) {
      context.strokeStyle = `rgba(255,255,255,${0.025 + ring * 0.003})`;
      context.lineWidth = 2 + ring * 0.6;
      context.beginPath();
      const cx = width * (0.2 + ((index * 0.17 + ring * 0.037) % 0.65));
      const cy = height * (0.15 + ((index * 0.11 + ring * 0.061) % 0.7));
      context.ellipse(cx, cy, 80 + ring * 31, 35 + ring * 18, index * 0.36 + ring * 0.05, 0, Math.PI * 2);
      context.stroke();
    }

    context.globalCompositeOperation = 'source-over';
    const shade = context.createLinearGradient(0, height * 0.35, 0, height);
    shade.addColorStop(0, 'rgba(0,0,0,0)');
    shade.addColorStop(1, 'rgba(0,0,0,0.72)');
    context.fillStyle = shade;
    context.fillRect(0, 0, width, height);

    context.fillStyle = 'rgba(255,255,255,0.72)';
    context.font = '500 20px "DM Mono", monospace';
    context.letterSpacing = '3px';
    context.fillText(`TF—0${index + 1}  ${project.kicker.toUpperCase()}`, 54, 62);
    context.textAlign = 'right';
    context.fillText('SELECTED EXPERIMENT · 2026', width - 54, 62);
    context.textAlign = 'left';

    context.strokeStyle = 'rgba(255,255,255,0.58)';
    context.lineWidth = 1;
    context.beginPath();
    context.moveTo(54, height - 188);
    context.lineTo(width - 54, height - 188);
    context.stroke();

    context.fillStyle = '#ffffff';
    context.font = '500 112px Arial, sans-serif';
    context.letterSpacing = '-6px';
    context.fillText(project.name, 50, height - 72);
    context.font = '400 22px Arial, sans-serif';
    context.letterSpacing = '0';
    context.fillStyle = 'rgba(255,255,255,0.82)';
    context.fillText(project.description, 56, height - 32);

    const texture = new THREE.CanvasTexture(art);
    texture.colorSpace = THREE.SRGBColorSpace;
    texture.anisotropy = 4;
    texture.needsUpdate = true;
    return texture;
  }
</script>

<canvas bind:this={canvas} aria-label="Drag to explore trustfall projects"></canvas>
<p class="renderer-label">{rendererType}</p>

<style>
  canvas {
    position: fixed;
    inset: 0;
    z-index: 0;
    width: 100%;
    height: 100%;
    touch-action: none;
    cursor: grab;
  }

  .renderer-label {
    position: fixed;
    right: 1.4rem;
    top: 4.6rem;
    z-index: 3;
    margin: 0;
    color: rgba(255, 255, 255, 0.58);
    font: 0.6rem/1 'DM Mono', monospace;
    letter-spacing: 0.12em;
    pointer-events: none;
    text-transform: uppercase;
  }

  @media (max-width: 640px) {
    .renderer-label {
      top: 3.8rem;
      right: 1rem;
    }
  }
</style>
