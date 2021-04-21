<template>
  <div>
    <canvas ref="webgl" class="webgl"></canvas>
  </div>
</template>
<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import * as dat from "dat.gui";
// import { gsap } from "gsap/all";


export default {
  data() {
    return {
      raycaster: null,
      mouse: {x: 0, y: 0},
      font: null,
      terrainGLTFObject: null,
      sceneGroup: null,
      settings: {
        showAnimation: false,
        rotationSpeed: 0.1,
      },
      terrainObject: null,

      sizes: {
        width: window.innerWidth,
        height: window.innerHeight,
      },
    
      // loaders
      gltfLoader: null,
      fontLoader: null,
      textureLoader: null,

      // debug
      gui: null,

      // canvas
      canvas: null,

      // scene
      scene: null,

      // camera
      camera: null,

      // controls
      controls: null,

      // renderer
      renderer: null,

      // clock
      clock: null,
    };
  },
  methods: {
    setCanvas() {
      this.canvas = document.querySelector("canvas.webgl");
    },
    createRenderer() {
      this.renderer = new THREE.WebGLRenderer({
        canvas: this.canvas,
      });
      this.renderer.setSize(this.sizes.width, this.sizes.height);
      this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    },
    createControls() {
      this.controls = new OrbitControls(this.camera, this.canvas);
      this.controls.enableDamping = true;
    },
    createCamera() {
      this.camera = new THREE.PerspectiveCamera(
        75,
        this.sizes.width / this.sizes.height,
        0.1,
        1000
      );
      this.camera.position.x = 15;
      this.camera.position.y = 8;
      this.camera.position.z = 8;
    },
    createScene() {
      this.scene = new THREE.Scene();
    },
    createDebugGUI() {
      this.gui = new dat.GUI();
    },
    createLoaders() {
      this.gltfLoader = new GLTFLoader();
      this.fontLoader = new THREE.FontLoader();
      this.textureLoader = new THREE.TextureLoader();
    },
    createClock() {
      this.clock = new THREE.Clock();
    },
    createRaycaster() {
      this.raycaster = new THREE.Raycaster();
    },
    createAxesHelper() {
      const axesHelper = new THREE.AxesHelper(10);
      this.scene.add(axesHelper);
    },
    addEventListeners() {
          window.addEventListener('dblclick', this.onMouseDoubleClick, false)
          window.addEventListener('click', this.onMouseClick, false)
            window.addEventListener("resize", this.handleResize);
          window.addEventListener('mousemove', this.onMouseMove)
    },
    initializeComponents() {
      this.setCanvas();
      this.createRenderer();
      this.createScene();
      this.createCamera();
      this.createControls();
      this.createDebugGUI();
      this.createLoaders();
      this.createAxesHelper();
      this.createRaycaster()
      this.createClock();
    },

    async load() {
      this.font = await this.loadFont("./static/fonts/helvetiker_regular.typeface.json");
      this.terrainGLTFObject = await this.loadTerrainObject();
    },
      loadFont(url) {
      return new Promise((resolve, reject) => {
        this.fontLoader.load(
          url,
          (texture) => resolve(texture),
          null,
          (err) => {
            reject(new Error(`Could not load texture ${url}:\n${err}`));
          }
        );
      });
    },
    async loadTerrainObject() {
           return new Promise((resolve) => {
        this.gltfLoader.load(
          "./static/models/Terrain/terrain.gltf",
          (gltf) => resolve(gltf),
          null,
          (error) => {
            console.log(`ERROR loadTerrainObject ${error}`);
            resolve();
          }
        );
      });   
    },
    displayTerrain(){
        console.log(this.terrainGLTFObject);
        this.terrainObject = this.terrainGLTFObject.scene;
        this.terrainObject.position.set(0,0,0)
        this.sceneGroup.add(this.terrainObject)
    },
    displayScene() {
    this.displayTerrain()
      this.scene.add(this.sceneGroup);
    },
    addDebugGUI() {
      // earth rotation
    //   this.gui.add(this.terrainObject.rotation, "x", -2, 2, 0.01);
    //   this.gui.add(this.terrainObject.rotation, "y", -2, 2, 0.01);
    //   this.gui.add(this.terrainObject.rotation, "z", -2, 2, 0.01);

    //   // animate earth rotation
    //   this.gui.add(this.settings, "showAnimation");

    //   // rotation speed
    //   this.gui.add(this.settings, "rotationSpeed", 0, 1, 0.01);
    },

    setLights() {
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
      this.gui.add(ambientLight, "visible").name("Ambient light visible");

      const pointLight = new THREE.PointLight(0xffffff, 1, 10000);
      pointLight.position.set(100, 100, 100);

      const pointLight2 = new THREE.PointLight(0xffffff, 1, 10000);
      pointLight2.position.set(-50, -50, -50);

      const pointLight3 = new THREE.PointLight(0xffffff, 1, 10000);
      pointLight3.position.set(0, 100, 0);

      const pointLight4 = new THREE.PointLight(0xffffff, 1, 10000);
      pointLight4.position.set(20, -20, 20);
      this.scene.add(
        ambientLight,
        pointLight,
        pointLight2,
        pointLight3,
        pointLight4
      );
    },
    tick() {
        // const elapsedTime = this.clock.getElapsedTime();

      // Update this.controls
      this.controls.update();

      // Render
      this.renderer.render(this.scene, this.camera);


      // Call tick again on the next frame
      window.requestAnimationFrame(this.tick);
    },
    async initializeSceneAndObjects() {
        this.sceneGroup = new THREE.Group()
        this.sceneGroup.userData.type = 'Planet'

        await this.load();
        this.displayScene();
        this.setLights()
        // this.addDebugGUI();
        this.tick();
    },
    async initialize() {
      
      this.initializeComponents();
      await this.initializeSceneAndObjects();

      this.addEventListeners();
    },
    handleResize() {
      // Update sizes
      this.sizes.width = window.innerWidth;
      this.sizes.height = window.innerHeight;

      // Update this.camera
      this.camera.aspect = this.sizes.width / this.sizes.height;
      this.camera.updateProjectionMatrix();

      // Update this.renderer
      this.renderer.setSize(this.sizes.width, this.sizes.height);
      this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    },
  },
  async mounted() {
    await this.initialize();
     
  },
};
</script>
<style scoped>
.webgl {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
}
</style>