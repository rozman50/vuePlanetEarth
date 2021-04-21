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
import config from "@/config.js";
import { gsap } from "gsap/all";

export default {
  data() {
    return {
      raycaster: null,
      mouse: {x: 0, y: 0},
      font: null,
      earthGLTFObject: null,
      sceneGroup: null,
      locationsGroup: null,
      planetEarthProperties: {
        boundingBox: null,
        radius: null,
      },
      hoveredLocation: null,
      settings: {
        showAnimation: false,
        rotationSpeed: 0.1,
      },
      earthObject: null,

      sizes: {
        width: window.innerWidth,
        height: window.innerHeight,
      },
      locationsOnEarth: [
        {
          name: "London",
          lat: 51.510133985168,
          lon: -0.1284266805390657,
        },
        {
          name: "Port Elizabeth ",
          lat: -33.946933946818056,
          lon: 25.565079089583335,
        },
        {
          name: "WashingtonDC",
          lat: 38.93773016201661,
          lon: -77.0427662913308,
        },
        {
          name: "Isla Hermite",
          lat: -55.88355403043114,
          lon: -67.58241237580057,
        },
        {
          name: "Tokio",
          lat: 35.680843670393514,
          lon: 139.7453946425056,
        },
        {
          name: "Riad",
          lat: 24.70830406500257,
          lon: 46.67457259362854,
        },
        {
          name: "Canbera",
          lat: -35.281794564002034,
          lon: 149.1287755452682,
        },
        {
          name: "Brasilia",
          lat: -15.79448410058518,
          lon: -47.88403962195319,
        },
        {
          name: "Testna afrika",
          lat: 0.6629132055499094 * (180 / Math.PI),
          lon: -6.037743742024168 * (180 / Math.PI),
        },
      ],
      raycasterCounter: 0,

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
    async loadEarthObject() {
      return new Promise((resolve) => {
        this.gltfLoader.load(
          "./static/models/earth/Earth.gltf",
          // "./static/models/planet_earth/scene.gltf",
          (gltf) => resolve(gltf),
          null,
          (error) => {
            console.log(`ERROR loadEarthObject ${error}`);
            resolve();
          }
        );
      });
    },
    async load() {
      this.font = await this.loadFont(
        "./static/fonts/helvetiker_regular.typeface.json"
      );
      this.earthGLTFObject = await this.loadEarthObject();
    },
    getEarthBoundingBox() {
      var earthBoundingBox = new THREE.Box3().setFromObject(this.earthObject);
      const size = new THREE.Vector3();
      earthBoundingBox.getSize(size);
      return size;
    },
    getEarthRadius() {
      const bb = this.getEarthBoundingBox();
      const radius = bb.x / 2;
      return radius;
    },
    displayEarth() {
      const scale = 10;
      this.earthObject = this.earthGLTFObject.scene;
      this.earthObject.scale.set(scale, scale, scale);
      this.earthObject.position.set(0, 0, 0)

      this.sceneGroup.add(this.earthObject);
      this.setPlanetEarthProperties();
    },
    setPlanetEarthProperties() {
      this.planetEarthProperties.boundingBox = this.getEarthBoundingBox();
      this.planetEarthProperties.radius = this.getEarthRadius();
    },
    calculateCoordinatesFromLongAndLat(lat, lon, radius) {
      const phi = (90 - lat) * (Math.PI / 180);
      const theta = (lon + 180) * (Math.PI / 180);
      const x = -(radius * Math.sin(phi) * Math.cos(theta));
      const z = radius * Math.sin(phi) * Math.sin(theta);
      const y = radius * Math.cos(phi);

      return { x, y, z };
    },
    createLocationTextMesh(text, x, y, z, color = "orange") {
      const fontProperties = {
        font: this.font,
        size: 0.5,
        height: 0.2,
        curveSegments: 2,
        bevelEnabled: true,
        bevelThickness: 0.03,
        bevelSize: 0.02,
        bevelOffset: 0,
        bevelSegments: 4,
      };
      const textMaterial = new THREE.MeshBasicMaterial({ color: color });
      const textGeometry = new THREE.TextGeometry(text, fontProperties);
      const textMesh = new THREE.Mesh(textGeometry, textMaterial);
      const textPositionMultiplier = 1.05;
      textMesh.position.set(
        x * textPositionMultiplier,
        y * textPositionMultiplier,
        z * textPositionMultiplier
      );

      // text gleda v sredino zemlje, ampak je zrcalno obrnjen
      textMesh.lookAt(this.earthObject.position);
      textMesh.scale.x *= -1;

      textGeometry.computeBoundingBox();
      textGeometry.center();
      return textMesh;
    },
    createLocationGroup(name, lat, lon, color = 0xffab00, addPin = false) {
      const radius = this.planetEarthProperties.radius;
      const { x, y, z } = this.calculateCoordinatesFromLongAndLat(
        lat,
        lon,
        radius
      );

      const textMesh = this.createLocationTextMesh(name, x, y, z, color);
      textMesh.userData.isLocationName = true;

      var textBoundingBox = new THREE.Box3().setFromObject(textMesh);
      const size = new THREE.Vector3();
      textBoundingBox.getSize(size);

      const planeMaterial = new THREE.MeshBasicMaterial({
        visible: true,
        color: color,
      });
      const planeGeometry = new THREE.PlaneGeometry(
        Math.max(size.x, size.y, size.z),
        1
      );
      const planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
      planeMesh.position.set(x, y, z);
      planeMesh.lookAt(
        new THREE.Vector3(
          this.earthObject.position.x,
          this.earthObject.position.y,
          this.earthObject.position.z
        )
      );
      planeMesh.scale.z *= -1; // obrni, da gleda navzven
      planeMesh.userData.isRaycastHitObject = true;

      const locationGroup = new THREE.Group();
      locationGroup.userData.type = "Location";
      locationGroup.add(textMesh);
      locationGroup.add(planeMesh);

      if (addPin) {
        const pinMaterial = new THREE.MeshBasicMaterial({ color: color });
        const pinGeometry = new THREE.SphereGeometry(0.2, 10, 10);

        const pinMesh = new THREE.Mesh(pinGeometry, pinMaterial);

        pinMesh.position.x = x;
        pinMesh.position.y = y;
        pinMesh.position.z = z;

        locationGroup.add(pinMesh);
      }

      return locationGroup;
    },
    rotateEarthToGreenWichAndEquator() {
      this.earthObject.rotation.x = -0.13;
      this.earthObject.rotation.y =  1.64;
      this.earthObject.rotation.z =  0.14;
    },
    displayLocationsOnEarth() {
      for (const location of this.locationsOnEarth) {
        const locationGroup = this.createLocationGroup(
          location.name,
          location.lat,
          location.lon,
          config.locationText.colorHex
        );

        this.locationsGroup.add(locationGroup);
      }
      this.sceneGroup.add(this.locationsGroup);
    },
    displayScene() {
      this.displayEarth();
      this.displayLocationsOnEarth();
      this.rotateEarthToGreenWichAndEquator();

      this.scene.add(this.sceneGroup);
    },
    addDebugGUI() {
      // earth rotation
      let rotationFolder = this.gui.addFolder('Rotation')
      rotationFolder.add(this.earthObject.rotation, "x", -2, 2, 0.01);
      rotationFolder.add(this.earthObject.rotation, "y", -2, 2, 0.01);
      rotationFolder.add(this.earthObject.rotation, "z", -2, 2, 0.01);

      let positionFolder = this.gui.addFolder('Position')
      positionFolder.add(this.earthObject.position, "x", -100, 100, 1);
      positionFolder.add(this.earthObject.position, "y", -100, 100, 1);
      positionFolder.add(this.earthObject.position, "z", -100, 100, 1);

      // animate earth rotation
      // this.gui.add(this.settings, "showAnimation");

      // rotation speed
      // this.gui.add(this.settings, "rotationSpeed", 0, 1, 0.01);
    },
    getLocationGroupsForRaycasting() {
      var objs = [];
      /**
       * this.locationsGroup: Array of all locations
       * LocationGroup: is Group of Text and Plane (plane has userData.isRaycastHitObject property)
       */
      for (const lg1 of this.locationsGroup.children) {
        for (const lg2 of lg1.children) {
          if (lg2.userData.isRaycastHitObject) {
            objs.push(lg2);
          }
        }
      }
      return objs;
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

    onMouseMove(event) {
      this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      this.mouse.y = - (event.clientY / window.innerHeight) * 2 + 1;
    },

    onMouseDoubleClick(event) {

    this.mouse.x = (event.clientX / this.sizes.width) * 2 - 1;
    this.mouse.y = - (event.clientY / this.sizes.height) * 2 + 1;

    // update picking ray
    this.raycaster.setFromCamera(this.mouse, this.camera)

    const intersects = this.raycaster.intersectObjects(this.sceneGroup.children, true)

    if (!intersects.length) return

    var locationName = prompt("Vpiši ime lokacije")

    const intersectObject = intersects[0] // get object that was clicked (water or terrain)
    const clickPosition = intersectObject.point  // get Vector3 click position
    const { lat, lon } = this.vector3toLonLat(new THREE.Vector3(clickPosition.x, clickPosition.y, clickPosition.z)) // convert xyz to lat,lon

    const locGroup = this.createLocationGroup(locationName, lat, lon, 0x00aaf5)

    this.locationsGroup.add(locGroup)
},

vector3toLonLat(vector3) // https://gist.github.com/nicoptere/2f2571db4b454bb18cd9
{

    vector3.normalize();

    //longitude = angle of the vector around the Y axis
    //-( ) : negate to flip the longitude (3d space specific )
    //- PI / 2 to face the Z axis
    // var lon = -( Math.atan2( -vector3.z, -vector3.x ) ) - Math.PI / 2; // FROM
    var lon = - Math.atan2(-vector3.z, -vector3.x) - Math.PI;


    //to bind between -PI / PI
    // if( lon < - Math.PI )lon += Math.PI * 2; // FROM 
    if (lon > Math.PI) lon -= Math.PI * 2;

    //latitude : angle between the vector & the vector projected on the XZ plane on a unit sphere

    //project on the XZ plane
    var p = new THREE.Vector3(vector3.x, 0, vector3.z);
    //project on the unit sphere
    p.normalize();

    //commpute the angle ( both vectors are normalized, no division by the sum of lengths )
    var lat = Math.acos(p.dot(vector3));

    //invert if Y is negative to ensure teh latitude is comprised between -PI/2 & PI / 2
    if (vector3.y < 0) lat *= -1;

    lon = lon * (180 / Math.PI)
    lat = lat * (180 / Math.PI)

    return { lat, lon };

},

onMouseClick() {
    
    const objectsToRaycast = this.getLocationGroupsForRaycasting()
    objectsToRaycast.map(o => o.visible = false)

    // update picking ray
    this.raycaster.setFromCamera(this.mouse, this.camera)

    const intersects = this.raycaster.intersectObjects(objectsToRaycast)

    if (!intersects.length) return
    console.log("click");
    console.log(intersects);
},


    checkRaycasterHitsOnLocations() {
      // update the picking ray with the this.camera and mouse position
      this.raycaster.setFromCamera(this.mouse, this.camera);

      const objectsToRaycast = this.getLocationGroupsForRaycasting();
      objectsToRaycast.map((o) => (o.visible = false));

      const intersects = this.raycaster.intersectObjects(objectsToRaycast);

      document.body.style.cursor = "auto";

      if (intersects.length > 0) {
        document.body.style.cursor = "pointer";
        const textMesh = intersects[0].object.parent.children.find(
          (c) => c.userData.isLocationName
        );

        if (this.hoveredLocation !== textMesh) {
          this.hoveredLocation = textMesh;
          gsap.to(this.hoveredLocation.scale, {
            x: -1.1,
            y: 1.1,
            duration: 0.2,
          });

          const rgbHoveredColor = new THREE.Color(
            config.locationText.hoveredColorHex
          );
          gsap.to(this.hoveredLocation.material.color, {
            r: rgbHoveredColor.r,
            g: rgbHoveredColor.g,
            b: rgbHoveredColor.b,
            duration: 0.2,
          });
        }
      } else {
        if (this.hoveredLocation) {
          gsap.to(this.hoveredLocation.scale, {
            x: -0.9,
            y: 0.9,
            duration: 0.2,
          });

          const rgbTextColor = new THREE.Color(config.locationText.colorHex);
          gsap.to(this.hoveredLocation.material.color, {
            r: rgbTextColor.r,
            g: rgbTextColor.g,
            b: rgbTextColor.b,
            duration: 0.2,
          });
        }
        this.hoveredLocation = null;
      }
    },
    tick() {
        const elapsedTime = this.clock.getElapsedTime();
      if (this.settings.showAnimation && this.earthObject) {
        this.sceneGroup.rotation.y = -this.settings.rotationSpeed * elapsedTime;
      }

      if (this.raycasterCounter >= 10) {
        this.checkRaycasterHitsOnLocations();
        this.raycasterCounter = 0;
      }
      this.raycasterCounter++;

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
        this.locationsGroup = new THREE.Group()
        this.locationsGroup.userData.type = 'this.locationsGroup'

        await this.load();
        this.displayScene();
        this.setLights()
        this.addDebugGUI();
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