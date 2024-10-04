<script setup lang='ts'>

import { onMounted } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js';
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader.js';
import { Reflector } from 'three/addons/objects/Reflector.js';
import EventBus from '../EventBus';

// переменные размера окна
let wrapper3Dw:number = window.innerWidth;
let wrapper3Dh:number = window.innerHeight;

// переменные параметров двери
let offset:number = 0;
let delta:number = 0.05;
let textureName:string = 'doorTexture.jpg';

// создаём сцену
const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({antialias: true, powerPreference: 'high-performance'});
scene.background = new THREE.Color('rgb(0,0,0)');
renderer.setSize(wrapper3Dw,wrapper3Dh);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap; 
renderer.shadowMapEnabled = true;
renderer.shadowMapType = THREE.PCFSoftShadowMap;

// создаём камеру
const camera = new THREE.PerspectiveCamera(75,wrapper3Dw/wrapper3Dw,0.1,100);
const controls = new OrbitControls(camera,renderer.domElement);
camera.position.set(-2, 0, 15);
camera.rotation.set(0, -0.3, 0);
camera.aspect = wrapper3Dw / wrapper3Dh;
camera.updateProjectionMatrix();
renderer.setPixelRatio(window.devicePixelRatio); 

// создаём источник освещения
const pointColor = 0xffffff;
const pointIntensity:number = 1200;
const pointLight = new THREE.PointLight(pointColor,pointIntensity);
pointLight.position.set(0,5,10);
pointLight.rotation.x = -Math.PI*2.1;
pointLight.castShadow = true;
pointLight.shadow.mapSize.x = 1024;
pointLight.shadow.mapSize.y = 1024;
scene.add(pointLight);

const loader = new THREE.TextureLoader();
const woodMaterial = new THREE.MeshBasicMaterial({
  side: THREE.FrontSide,
  map: loader.load(textureName,(texture)=>{
    texture.repeat.set(1,1);
    texture.wrapS = THREE.ClampToEdgeWrapping;
    texture.wrapT = THREE.ClampToEdgeWrapping;
    texture.generateMipmaps = false;
    texture.minFilter = THREE.LinearFilter;
    texture.needsUpdate = true;
    texture.anisotropy =  16;
  }),
});

// создаём дверь

const doorElementGeometry1 = new THREE.BoxGeometry(5,10,0.2); 
const doorElementMaterial1 = woodMaterial.clone();
doorElementMaterial1.color = new THREE.Color('rgb(145,139,0)');
const doorElement1 = new THREE.Mesh(doorElementGeometry1,doorElementMaterial1); 
doorElement1.position.set(0,0,0.2);
scene.add(doorElement1);

const doorElementGeometry2 = new THREE.SphereGeometry(0.3); 
const doorElementMaterial2 = woodMaterial.clone();
doorElementMaterial2.color = new THREE.Color('rgb(145,95,0)');
const doorElement2 = new THREE.Mesh(doorElementGeometry2,doorElementMaterial2);
doorElement2.position.set(1.8,-1,0.5);
scene.add(doorElement2);

const doorElementGeometry3 = new THREE.BoxGeometry(4,4,0.1); 
const doorElementMaterial3 = woodMaterial.clone();
doorElementMaterial3.color = new THREE.Color('rgb(125,119,50)');
const doorElement3 = new THREE.Mesh(doorElementGeometry3,doorElementMaterial3);
doorElement3.position.set(0,2.5,0.3);
scene.add(doorElement3);

const doorElementGeometry4 = new THREE.BoxGeometry(4,2,0.1); 
const doorElementMaterial4 = woodMaterial.clone();
doorElementMaterial4.color = new THREE.Color('rgb(125,119,50)');
const doorElement4 = new THREE.Mesh(doorElementGeometry4,doorElementMaterial4); 
doorElement4.position.set(0,-3.5,0.3);
scene.add(doorElement4);

const doorElementGeometry5 = new THREE.BoxGeometry(5.4,10.2,0.2); 
const doorElementMaterial5 = woodMaterial.clone();
doorElementMaterial5.color = new THREE.Color('rgb(145,95,0)');
const doorElement5 = new THREE.Mesh(doorElementGeometry5,doorElementMaterial5); 
doorElement5.position.set(0,0.1,0.1);
scene.add(doorElement5);

// создаём зеркальный пол:

// создаём отражение
const geometry = new THREE.PlaneGeometry(100, 100, 64, 64);
let groundMirror = new Reflector(geometry, {
  clipBias: 0.03,
  textureWidth: wrapper3Dw * window.devicePixelRatio,
  textureHeight: wrapper3Dh * window.devicePixelRatio,
  color: 0xcccccc
});
groundMirror.position.y = -5;
groundMirror.rotateX(-Math.PI/2);
scene.add(groundMirror);

// создаём стекло
const geometryGlass = new THREE.PlaneGeometry(100, 100, 64, 64);
const materialGlass = new THREE.MeshPhysicalMaterial({  
 roughness: 0,
 transmission: 1,
 thickness: 2,
 side: THREE.DoubleSide,
 transparent: true,
 opacity: 0.8,
 clearcoat: 0,
 reflectivity: 0.1,
 specularIntensity: 0, 
 color: 0xcccccc
});
const floorGlass = new THREE.Mesh(geometryGlass, materialGlass);
floorGlass.position.y = -4.9;
floorGlass.rotateX(-Math.PI/2);
floorGlass.castShadow = true; 
floorGlass.receiveShadow = true;
scene.add(floorGlass);

// создаём стены
const wallGeometry = new THREE.PlaneGeometry(20, 12, 2, 2);
const wallMaterial = new THREE.MeshPhongMaterial({
  color: new THREE.Color('rgb(158,163,119)')
});
const wall = new THREE.Mesh(wallGeometry , wallMaterial);
wall.position.y = 1;
wall.castShadow = true; 
wall.receiveShadow = true;
scene.add(wall);

const wall2 = wall.clone();
wall2.rotation.y = Math.PI/2;
wall2.position.x = -10;
wall2.castShadow = true; 
wall2.receiveShadow = true;
scene.add(wall2);

const wall3 = wall.clone();
wall3.rotation.y = -Math.PI/2;
wall3.position.x = 10;
wall3.castShadow = true; 
wall3.receiveShadow = true;
scene.add(wall3);

// функция изменения размеров окна браузера
let resize = () => {
  camera.aspect = wrapper3Dw / wrapper3Dh;
  camera.updateProjectionMatrix();
  renderer.setSize(wrapper3Dw,wrapper3Dh);
  renderer.setPixelRatio(window.devicePixelRatio); 
}

// функция изменения размеров двери
let changeDoorSize = (dir:string):void => {
  if (dir == 'more') {
    offset += delta;
  } else if (dir == 'less') {
    offset -= delta;
  }  
  
  doorElement1.geometry.dispose();
  doorElement1.geometry = new THREE.BoxGeometry(5,10+offset,0.2);
  doorElement1.position.set(0,(10+offset-10)/2,0.2);
  
  doorElement3.geometry.dispose();
  doorElement3.geometry = new THREE.BoxGeometry(4,4+offset,0.1);
  doorElement3.position.set(0,2.5+(4+offset-4)/2,0.3);
  
  doorElement5.geometry.dispose();
  doorElement5.geometry = new THREE.BoxGeometry(5.4,10.2+offset,0.2);
  doorElement5.position.set(0,0.1+(10.2+offset-10.2)/2,0.1);
}

let animate = () => {
  renderer.render(scene,camera);
  requestAnimationFrame(animate);
}; animate();

EventBus.$on('event name', (data:any) => {
    changeDoorSize(data.size);      
});

onMounted(() => {
    document.querySelector('#scene')!.appendChild(renderer.domElement);
    window.addEventListener('resize',resize);
});

</script>

<template>
  <div id='scene'></div>
</template>
