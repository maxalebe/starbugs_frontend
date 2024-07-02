<script>
  import { onMount } from 'svelte';
  import axios from 'axios';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

  let scene, camera, renderer, controls;
  let axesScene, axesCamera, axesRenderer, axesHelper;
  let viewDistance = 175;
  let ringMesh;
  let starMeshes = [];

  function init() {
    scene = new THREE.Scene();
    scene.background = new THREE.Color("black");

    camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 20, 100);
    camera.lookAt(0, 0, 0);

    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableZoom = true;
    controls.enablePan = true;
    controls.dampingFactor = 0.05;

    axesScene = new THREE.Scene();
    axesCamera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 10);
    axesRenderer = new THREE.WebGLRenderer({ antialias: true });
    axesRenderer.setSize(100, 100);
    axesRenderer.domElement.style.position = 'absolute';
    axesRenderer.domElement.style.top = '10px';
    axesRenderer.domElement.style.right = '10px';
    document.body.appendChild(axesRenderer.domElement);

    axesHelper = new THREE.AxesHelper(1);
    axesScene.add(axesHelper);

    axesRenderer.domElement.addEventListener('mousedown', onAxesMouseDown, false);

    const ringGeometry = new THREE.RingGeometry(4, 4.2, 32);
    const ringMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00, side: THREE.DoubleSide });
    ringMesh = new THREE.Mesh(ringGeometry, ringMaterial);
    ringMesh.position.set(0, 0, 0);
    ringMesh.rotation.x = Math.PI / 2;
    scene.add(ringMesh);

    showStars();

    // Add mouse move listener for hovering over stars
    window.addEventListener('mousemove', onMouseMove, false);
  }

  function onAxesMouseDown(event) {
    event.preventDefault();
    const rect = axesRenderer.domElement.getBoundingClientRect();
    const mouse = new THREE.Vector2();
    mouse.x = (event.clientX - rect.left) / rect.width * 2 - 1;
    mouse.y = -(event.clientY - rect.top) / rect.height * 2 + 1;
    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(mouse, axesCamera);
    const intersects = raycaster.intersectObject(axesHelper);

    if (intersects.length > 0) {
      const point = intersects[0].point;
      controls.target.copy(point);
      controls.update();
    }
  }

  async function showStars() {
    try {
      const response = await axios.get('https://newstar-api.maximebeck.de/star-data');
      const stars = response.data;

      scene.children = scene.children.filter(child => !(child instanceof THREE.Mesh) || child === ringMesh);
      starMeshes = [];

      if (viewDistance <= 0 || stars.length < 1) return;

      stars.forEach((star) => {
        const geometry = new THREE.SphereGeometry(0.2, 16, 16);
        const material = new THREE.ShaderMaterial({
          uniforms: {
            time: { value: Math.random() * 1000 },
            hover: { value: false }
          },
          vertexShader: `
            varying vec3 vNormal;
            void main() {
              vNormal = normalize(normalMatrix * normal);
              gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
          `,
          fragmentShader: `
            uniform float time;
            uniform bool hover;
            varying vec3 vNormal;
            void main() {
              float intensity = sin(time + length(vNormal) * 10.0) * 0.5 + 0.5;
              vec3 color = vec3(intensity);
              if (hover) {
                color = vec3(1.0, 1.0, 0.0);
              } else if (${star.wikiUrl ? 'true' : 'false'}) {
                color = vec3(0.0, 0.0, 1.0);
              }
              gl_FragColor = vec4(color, 1.0);
            }
          `,
          transparent: true
        });
        const starMesh = new THREE.Mesh(geometry, material);
        starMesh.position.set(star.x0, star.y0, star.z0);
        starMesh.userData = { wikiUrl: star.wikiUrl };
        scene.add(starMesh);
        starMeshes.push(starMesh);
      });
    } catch (error) {
      console.error('Error fetching star data:', error);
    }
  }

  function onMouseMove(event) {
    const mouse = new THREE.Vector2(
      (event.clientX / window.innerWidth) * 2 - 1,
      -(event.clientY / window.innerHeight) * 2 + 1
    );
    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(starMeshes);

    starMeshes.forEach(starMesh => {
      starMesh.material.uniforms.hover.value = false;
    });

    if (intersects.length > 0) {
      const intersectedStar = intersects[0].object;
      if (intersectedStar.userData.wikiUrl) {
        intersectedStar.material.uniforms.hover.value = true;
      }
    }
  }

  onMount(async () => {
    init();
    await showStars();
    animate();
  });

  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    starMeshes.forEach(starMesh => {
      starMesh.material.uniforms.time.value += 0.02;
    });
    renderer.render(scene, camera);
    axesRenderer.render(axesScene, axesCamera);
  }

  function updateViewDistance(event) {
    viewDistance = event.target.value;
    console.log("Updating view distance to:", viewDistance);
    camera.far = Math.min(Number(viewDistance), 350);
    camera.updateProjectionMatrix();
    if (viewDistance > 0) {
      showStars();
    } else {
      scene.children = scene.children.filter(child => !(child instanceof THREE.Mesh));
      scene.add(ringMesh);
    }
  }

  function resetCameraPosition() {
    const duration = 1000;
    const startPosition = camera.position.clone();
    const startTarget = controls.target.clone();
    const endPosition = new THREE.Vector3(0, 20, 100);
    const endTarget = new THREE.Vector3(0, 0, 0);

    const startTime = performance.now();

    function animateReset() {
      const elapsedTime = performance.now() - startTime;
      const t = Math.min(elapsedTime / duration, 1);
      camera.position.lerpVectors(startPosition, endPosition, t);
      controls.target.lerpVectors(startTarget, endTarget, t);
      controls.update();

      if (t < 1) {
        requestAnimationFrame(animateReset);
      }
    }

    animateReset();
  }

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
</script>

<main>
  <label for="viewDistance">View Distance:</label>
  <input type="range" id="viewDistance" min="20" max="350" value="175" on:input={updateViewDistance} />
  <input type="number" id="viewDistanceInput" min="20" max="350" value="175" on:input={updateViewDistance} style="width: 60px; margin-left: 10px;">
  <button on:click={resetCameraPosition} style="margin-left: 10px;">Reset Camera</button>
</main>

<style>
  * {
    box-sizing: border-box;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }

  body {
    margin: 0;
    overflow: hidden;
  }

  main {
    position: absolute;
    top: 10px;
    left: 10px;
    background: rgba(255, 255, 255, 0.8);
    padding: 10px;
    border-radius: 5px;
    display: flex;
    align-items: center;
  }

  button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 3px;
    cursor: pointer;
  }

  button:hover {
    background-color: #0056b3;
  }
</style>
