<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three';

  export let metalType    = 'gold';
  export let diamondType  = 'white';
  export let finish       = 'polished';
  export let mood         = 'studio';
  export let diamondSize  = 1.0;
  export let sparkle      = true;
  export let isLoaded     = false;

  let canvas;

  let ringMats      = [];
  let connectorMats = [];
  let diamondMat    = null;
  let diamondNode   = null;
  let diamondVisual = null;
  let sparkLights   = [];

  // The diamond mesh origin in Blender sits slightly inside the gem.
  // Multiplying every scale by 1.15 ensures the minimum slider value
  // (0.6 × 1.15 = 0.69) always keeps the gem flush with its setting.
  const DIAMOND_BASE_SCALE = 1.15;

  const METALS = {
    gold:     { color: new THREE.Color(0xC8A858), metalness: 0.98 },
    silver:   { color: new THREE.Color(0xD4D4D4), metalness: 0.99 },
    rose:     { color: new THREE.Color(0xC4816A), metalness: 0.97 },
    platinum: { color: new THREE.Color(0xE0DED8), metalness: 0.99 },
  };

  const FINISH_ROUGHNESS = { polished: 0.08, brushed: 0.45, matte: 0.78 };

  const DIAMOND_TINTS = {
    white:     new THREE.Color(0xFFFFFF),
    blue:      new THREE.Color(0xD8F0FF),
    lightblue: new THREE.Color(0xE8F8FF),
    pink:      new THREE.Color(0xFFECF0),
    yellow:    new THREE.Color(0xFFFBE0),
  };

  const MOODS = {
    studio:      { hdr: 'https://dl.polyhaven.org/file/ph-assets/HDRIs/hdr/1k/studio_small_09_1k.hdr',  exposure: 0.85 },
    golden:      { hdr: 'https://dl.polyhaven.org/file/ph-assets/HDRIs/hdr/1k/golden_bay_1k.hdr',       exposure: 1.0  },
    twilight:    { hdr: 'https://dl.polyhaven.org/file/ph-assets/HDRIs/hdr/1k/kloppenheim_06_1k.hdr',   exposure: 0.7  },
    candlelight: { hdr: 'https://dl.polyhaven.org/file/ph-assets/HDRIs/hdr/1k/studio_small_09_1k.hdr',  exposure: 0.6  },
  };

  $: applyMetal(metalType, finish);
  $: applyFinish(finish, metalType);
  $: applyDiamond(diamondType);
  $: applyDiamondSize(diamondSize);
  $: applySparkle(sparkle);

  function applyMetal(type) {
    const m = METALS[type]; if (!m) return;
    const rough = FINISH_ROUGHNESS[finish] ?? 0.08;
    for (const mat of [...ringMats, ...connectorMats]) {
      mat.color.copy(m.color);
      mat.metalness = m.metalness;
      mat.roughness = rough;
      mat.needsUpdate = true;
    }
  }

  function applyFinish(f) {
    const rough = FINISH_ROUGHNESS[f] ?? 0.08;
    const m = METALS[metalType];
    for (const mat of [...ringMats, ...connectorMats]) {
      mat.roughness = rough;
      if (m) mat.metalness = f === 'matte' ? 0.6 : m.metalness;
      mat.needsUpdate = true;
    }
  }

  function applyDiamond(type) {
    if (!diamondMat) return;
    const col = DIAMOND_TINTS[type]; if (!col) return;
    diamondMat.color.copy(col);
    if (type !== 'white') {
      diamondMat.emissive.set(col.r * 0.04, col.g * 0.04, col.b * 0.04);
    } else {
      diamondMat.emissive.set(0, 0, 0);
    }
    diamondMat.needsUpdate = true;
  }

  function applyDiamondSize(s) {
    if (!diamondVisual) return;
    diamondVisual.scale.setScalar(s * DIAMOND_BASE_SCALE);
  }

  function applySparkle(on) {
    for (const l of sparkLights) l.visible = on;
  }

  onMount(async () => {
    const { GLTFLoader }    = await import('three/examples/jsm/loaders/GLTFLoader.js');
    const { OrbitControls } = await import('three/examples/jsm/controls/OrbitControls.js');
    const { RGBELoader }    = await import('three/examples/jsm/loaders/RGBELoader.js');

    const scene = new THREE.Scene();
    scene.background = null;

    const rect = canvas.getBoundingClientRect();
    const camera = new THREE.PerspectiveCamera(36, rect.width / rect.height, 0.01, 100);
    camera.position.set(0, 0.4, 2.6);

    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
    renderer.setSize(rect.width, rect.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.outputColorSpace = THREE.SRGBColorSpace;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.toneMappingExposure = MOODS[mood].exposure;

    const rgbeLoader = new RGBELoader();
    let currentHdr = null;
    function loadHdr(url, exposure) {
      rgbeLoader.load(url, (tex) => {
        tex.mapping = THREE.EquirectangularReflectionMapping;
        if (currentHdr) currentHdr.dispose();
        currentHdr = tex;
        scene.environment = tex;
        renderer.toneMappingExposure = exposure;
      });
    }
    loadHdr(MOODS[mood].hdr, MOODS[mood].exposure);

    const ambientLight = new THREE.AmbientLight(0xfff8f0, 0.12);
    scene.add(ambientLight);
    const keyLight = new THREE.DirectionalLight(0xfff4e0, 0.5);
    keyLight.position.set(-2, 4, 2);
    scene.add(keyLight);
    const fillLight = new THREE.DirectionalLight(0xf0f4ff, 0.25);
    fillLight.position.set(3, 1, -2);
    scene.add(fillLight);

    const sparkColors = [0xffffff, 0xfff0d0, 0xd0f0ff];
    for (let i = 0; i < 3; i++) {
      const sl = new THREE.PointLight(sparkColors[i], 0.8, 1.5);
      sl.visible = sparkle;
      scene.add(sl);
      sparkLights.push(sl);
    }

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping   = true;
    controls.dampingFactor   = 0.05;
    controls.autoRotate      = true;
    controls.autoRotateSpeed = 0.5;
    controls.minDistance     = 1.2;
    controls.maxDistance     = 5;
    controls.enablePan       = false;
    controls.minPolarAngle   = Math.PI * 0.15;
    controls.maxPolarAngle   = Math.PI * 0.8;

    const loader = new GLTFLoader();
    loader.load('/models/2ring.glb', (gltf) => {
      const model = gltf.scene;
      const box    = new THREE.Box3().setFromObject(model);
      const center = box.getCenter(new THREE.Vector3());
      const size   = box.getSize(new THREE.Vector3()).length();
      model.position.sub(center);
      model.scale.setScalar(2 / size);

      model.traverse((child) => {
        if (!child.isMesh) return;
        const mName = child.material?.name ?? '';

        if (mName === 'Ring.001') {
          const mat = new THREE.MeshStandardMaterial({
            color:           METALS[metalType].color.clone(),
            metalness:       METALS[metalType].metalness,
            roughness:       FINISH_ROUGHNESS[finish],
            envMapIntensity: 1.8,
          });
          child.material = mat;
          ringMats.push(mat);
        }

        if (mName === 'Diamond2.001') {
          const mat = new THREE.MeshPhysicalMaterial({
            color:              DIAMOND_TINTS[diamondType].clone(),
            metalness:          0,
            roughness:          0,
            transmission:       0.96,
            ior:                2.42,
            thickness:          0.6,
            envMapIntensity:    2.8,
            clearcoat:          1,
            clearcoatRoughness: 0,
            reflectivity:       1,
            specularIntensity:  1,
            specularColor:      new THREE.Color(0xffffff),
            transparent:        true,
            side:               THREE.DoubleSide,
            emissive:           new THREE.Color(0x000000),
            emissiveIntensity:  1,
          });
          child.material = mat;
          diamondMat = mat;

          const wrapper = new THREE.Group();
          child.parent.add(wrapper);
          wrapper.position.copy(child.position);
          wrapper.rotation.copy(child.rotation);
          child.position.set(0, 0, 0);
          child.rotation.set(0, 0, 0);
          wrapper.add(child);

          diamondNode   = wrapper;
          diamondVisual = child;
          child.scale.setScalar(diamondSize * DIAMOND_BASE_SCALE);
        }

        if (mName === 'Material.001') {
          const mat = new THREE.MeshStandardMaterial({
            color:           METALS[metalType].color.clone(),
            metalness:       METALS[metalType].metalness,
            roughness:       FINISH_ROUGHNESS[finish] + 0.04,
            envMapIntensity: 1.6,
          });
          child.material = mat;
          connectorMats.push(mat);
        }
      });

      scene.add(model);
      isLoaded = true;
    });

    const obs = new ResizeObserver(() => {
      const r = canvas.getBoundingClientRect();
      if (r.width === 0 || r.height === 0) return;
      camera.aspect = r.width / r.height;
      camera.updateProjectionMatrix();
      renderer.setSize(r.width, r.height);
    });
    obs.observe(canvas);

    let lastMood = mood;
    function checkMoodChange() {
      if (mood !== lastMood) {
        lastMood = mood;
        loadHdr(MOODS[mood].hdr, MOODS[mood].exposure);
        if (mood === 'candlelight') {
          ambientLight.color.set(0xff9944); ambientLight.intensity = 0.08;
          keyLight.color.set(0xff8822);
        } else if (mood === 'golden') {
          ambientLight.color.set(0xffd080); keyLight.color.set(0xffd080);
          ambientLight.intensity = 0.15;
        } else {
          ambientLight.color.set(0xfff8f0); keyLight.color.set(0xfff4e0);
          ambientLight.intensity = 0.12;
        }
      }
    }

    let animId;
    let t = 0;
    (function animate() {
      animId = requestAnimationFrame(animate);
      t += 0.016;
      checkMoodChange();
      if (sparkle && diamondNode) {
        const dp = new THREE.Vector3();
        diamondNode.getWorldPosition(dp);
        sparkLights[0].position.set(dp.x + Math.cos(t * 1.3) * 0.5,     dp.y + Math.sin(t * 0.8) * 0.3 + 0.2, dp.z + Math.sin(t * 1.3) * 0.5);
        sparkLights[1].position.set(dp.x + Math.cos(t * 0.9 + 2) * 0.4, dp.y + 0.1,                            dp.z + Math.sin(t * 0.9 + 2) * 0.4);
        sparkLights[2].position.set(dp.x + Math.cos(t * 1.1 + 4) * 0.35,dp.y - 0.1,                            dp.z + Math.sin(t * 1.1 + 4) * 0.35);
      }
      controls.update();
      renderer.render(scene, camera);
    })();

    return () => {
      obs.disconnect();
      cancelAnimationFrame(animId);
      renderer.dispose();
    };
  });
</script>

<canvas bind:this={canvas} style="width:100%;height:100%;display:block;"></canvas>