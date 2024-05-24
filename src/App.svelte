<script>
  import { onMount } from 'svelte';
  import axios from 'axios';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  
  let scene, camera, renderer, controls;
  let viewDistance = 175; // Initial view distance set to 175, max set to 350

  function init() {
    // Create the scene
    scene = new THREE.Scene();
    scene.background = new THREE.Color("black"); // Set background to black
  
    // Create the camera
    camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, viewDistance);
    
    // Set camera position to look at the origin (planet) and down
    camera.position.set(0, 20, 100); // Adjust position as needed
    camera.lookAt(0, 0, 0); // Look at the origin (planet)
  
    // Create the renderer
    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);
  
    // Add orbit controls
    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableZoom = true;
    controls.enablePan = true;
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
  }
  
  async function showStars() {
    const response = await axios.get('https://newstar-api.maximebeck.de/star-data');
    const stars = response.data;

    // Clear existing stars from the scene
    scene.children = scene.children.filter(child => !(child instanceof THREE.Mesh));

    if (viewDistance <= 0 || stars.length < 1) return; // Early exit if view distance is zero or no stars found

    stars.forEach((star) => {
      const geometry = new THREE.SphereGeometry(0.2, 16, 16);
      const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
      const starMesh = new THREE.Mesh(geometry, material);
      starMesh.position.set(star.x0, star.y0, star.z0);
      scene.add(starMesh);
    });
  }

  onMount(async () => {
    init();
    await showStars();
    animate();
  });
  
  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  }

  function updateViewDistance(event) {
    viewDistance = event.target.value;
    console.log("Updating view distance to:", viewDistance); // Debugging output
    camera.far = Math.min(Number(viewDistance), 350); // Ensure it's a number and capped at 350
    camera.updateProjectionMatrix();
    if (viewDistance > 0) {
      showStars(); // Update stars only if view distance is greater than zero
    } else {
      // Clear stars if view distance is zero
      scene.children = scene.children.filter(child => !(child instanceof THREE.Mesh));
    }
  }
</script>

<main>
  <label for="viewDistance">View Distance:</label>
  <input type="range" id="viewDistance" min="20" max="350" value="175" on:input={updateViewDistance} />
  <input type="number" id="viewDistanceInput" min="20" max="350" value="175" on:input={updateViewDistance} style="width: 60px; margin-left: 10px;">
</main>

<style>
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
  }
</style>
