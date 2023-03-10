<template>
    <div style="width: 100%;;height:100vh;" id="container"></div>
</template>
<script lang="ts" setup>
import { ref, reactive, onMounted, render } from 'vue'
import * as THREE from "three"
import { Mesh } from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";

let scene = null
let renderer = null
let camera = null
let controls = null

let gltfLoader = null
let dracoLoader = null

let figureGltf = null
let mixerScene = null

let clock = new THREE.Clock()
let previousTime = null

const dom = ref()

onMounted(() => {
    init()
    // addTestCube()
    addAirort()
    addLights()

    renderScene()
    addAxesHelper()
    window.addEventListener('resize', onWindowResize)


})

const init = () => {
    dom.value = document.getElementById('container')
    scene = new THREE.Scene()
    renderer = new THREE.WebGL1Renderer({
        antialias: true,
        // alpha: true,    // 画布是否包含alpha（透明度）缓冲区。默认值为false。
        logarithmicDepthBuffer: true,

    })

    let ambientLight = new THREE.AmbientLight(0x0c0c0c) // 创建环境光
    scene.add(ambientLight) // 将环境光添加到场景
    ambientLight.color = new THREE.Color(0x26E250) // 给环境光修改颜色
    ambientLight.visible = true //显示环境光

    renderer.outputEncoding = THREE.sRGBEncoding
    renderer.setSize(dom.value.offsetWidth, dom.value.offsetHeight)

    camera = new THREE.PerspectiveCamera(45, dom.value.offsetWidth / dom.value.offsetHeight, 0.1, 1000)

    //设置相机初始位置 
    // camera.position.set(3.03079142090544, 1.9848119701735485, 3.0720229175612066)
    camera.position.set(217.3103621558218, 126.41836032961118, 168.67448573029623)

    dom.value.appendChild(renderer.domElement)

    gltfLoader = new GLTFLoader();
    dracoLoader = new DRACOLoader();
    dracoLoader.setDecoderPath('examples/js/libs/draco/');
    gltfLoader.setDRACOLoader(dracoLoader);

    controls = new OrbitControls(camera, renderer.domElement)

    // 禁止场景向下拖拽
    // controls.maxPolarAngle = 0.4 * Math.PI;

}

//测试立方体
const addTestCube = () => {
    const boxGeometry = new THREE.BoxGeometry(1, 1, 1)
    const boxMaterial = new THREE.MeshBasicMaterial({
        color: 0x00ff00,
    })
    const cube = new Mesh(boxGeometry, boxMaterial)
    cube.position.set(1, 1, 1)
    scene.add(cube)
}

//添加模型
const addAirort = () => {
    //热气球缆车
    let balloonCableCar = new Promise((resolve) => {
        gltfLoader.load('/models/CesiumBalloon.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            // console.log("热气球缆车", gltf.animations)
            gltf.scene.translateY(55)
        })
    })

    //四翼桨机
    let fourWingsPropeller = new Promise((resolve) => {
        gltfLoader.load('/models/CesiumDrone.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);

            console.log("四翼桨机", gltf.animations)

            figureGltf = gltf
            mixerScene = new THREE.AnimationMixer(figureGltf.scene)

            gltf.scene.translateX(30)
            gltf.scene.translateY(30)
            gltf.scene.translateZ(30)

        })
    });

    //汽车
    let car = new Promise((resolve) => {
        gltfLoader.load('/models/CesiumMilkTruck.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            console.log("汽车", gltf.animations)

            gltf.scene.translateZ(40)


        })
    });

    //飞机
    let aircraft = new Promise((resolve) => {
        gltfLoader.load('/models/Cesium_Air.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            console.log("飞机", gltf.animations)

            figureGltf = gltf
            mixerScene = new THREE.AnimationMixer(figureGltf.scene)

            gltf.scene.translateX(0)
            gltf.scene.translateY(70)
            gltf.scene.translateZ(100)


        })
    });

    //人物
    let figure = new Promise((resolve) => {
        gltfLoader.load('/models/Cesium_Man.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);

            // figureGltf = gltf
            // mixerScene = new THREE.AnimationMixer(figureGltf.scene)

            gltf.scene.translateZ(20)
        })
    });

    //装甲车
    let armoredCar = new Promise((resolve) => {
        gltfLoader.load('/models/GroundVehicle.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            console.log("装甲车", gltf.animations)

            gltf.scene.translateZ(60)

        })
    });

    //哨塔
    let tower = new Promise((resolve) => {
        gltfLoader.load('/models/Wood_Tower.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            // console.log("哨塔", gltf.animations)
            gltf.scene.translateX(50)

        })
    });

    //ar-h470枪
    let spear = new Promise((resolve) => {
        gltfLoader.load('/models/ar-h470.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            // console.log("spear", gltf.animations)
            // gltf.scene.translateX(50)

        })
    });

    //原子之心机器人
    let atomic = new Promise((resolve) => {
        gltfLoader.load('/models/atomic.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            console.log("atomic", gltf.animations)
            // gltf.scene.translateX(50)

        })
    });

    //剑齿虎
    let saber = new Promise((resolve) => {
        gltfLoader.load('/models/saber.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            console.log("atomic", gltf.animations)
            // gltf.scene.translateX(50)

        })
    });

    //偷皮者
    let skin = new Promise((resolve) => {
        gltfLoader.load('/models/skin.glb', gltf => {
            gltf.scene.traverse(child => {
                if (child.isMesh) {
                    child.material.alphaTest = 1
                    child.material.side = THREE.DoubleSide;
                }
            })
            resolve(gltf.scene);
            console.log("atomic", gltf.animations)
            // gltf.scene.translateX(50)

        })
    });

    Promise.all([balloonCableCar, fourWingsPropeller, car, aircraft, figure, armoredCar, tower, spear, atomic, saber, skin]).then((resolve) => {
        scene.add(resolve[0])
        scene.add(resolve[1])
        scene.add(resolve[2])
        scene.add(resolve[3])
        scene.add(resolve[4])
        scene.add(resolve[5])
        scene.add(resolve[6])
        scene.add(resolve[7])
        scene.add(resolve[8])
        scene.add(resolve[9])
        scene.add(resolve[10])

        playSceneAnimation()

    }).catch(err => {
        console.log("添加模型抛错err", err)
    })
}

//播放场景动画
const playSceneAnimation = () => {
    const nLen = figureGltf.animations.length
    // console.log(nLen)
    for (let n = 0; n < nLen; ++n) {
        const action = mixerScene.clipAction(figureGltf.animations[n])
        action.paused = false
        action.play()
    }
}
//暂停场景动画
const stopSceneAnimation = () => {
    const nLen = figureGltf.animations.length
    // console.log(nLen)
    for (let n = 0; n < nLen; ++n) {
        const action = mixerScene.clipAction(figureGltf.animations[n])
        action.paused = true
    }
}

//添加环境光
const addLights = () => {
    let hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 1)
    scene.add(hemiLight)

    let directionalLight = new THREE.DirectionalLight(0xffffff, 0xffffff, 1)
    scene.add(directionalLight)
}


//绘制场景
function renderScene() {
    const elapsedTime = clock.getElapsedTime()
    const deltaTime = elapsedTime - previousTime
    previousTime = elapsedTime

    requestAnimationFrame(renderScene.bind(renderScene))
    renderer.render(scene, camera)

    controls.update()

    if (mixerScene) {
        mixerScene.update(deltaTime)
    }

    // console.log("相机位置", camera.position)
}

//场景自适应
const onWindowResize = () => {
    camera.aspect = dom.value.offsetWidth / dom.value.offsetHeight
    camera.updateProjectionMatrix()
    renderer.setSize(dom.value.offsetWidth, dom.value.offsetHeight)
}

//添加坐标轴
const addAxesHelper = () => {
    let axesHelper = new THREE.AxesHelper(300)
    axesHelper.position.set(0, 0, 0)
    scene.add(axesHelper)
}

</script>
<style></style>


