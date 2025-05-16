<template>
  <div ref="container" class="scene-container"></div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import * as THREE from 'three'
import { SVGLoader } from 'three/examples/jsm/loaders/SVGLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { Text } from 'troika-three-text'
import * as dat from 'dat.gui'

const container = ref(null)

onMounted(() => {
  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0xf0f0f0)

  const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000)
  camera.position.set(0, 0, 1000)

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  container.value.appendChild(renderer.domElement)

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  scene.add(new THREE.AmbientLight(0x888888))
  const dirLight = new THREE.DirectionalLight(0xffffff, 0.8)
  dirLight.position.set(100, 100, 100)
  scene.add(dirLight)

  const objectGroup = new THREE.Group()
  scene.add(objectGroup)

  const textGroup = new THREE.Group()
  scene.add(textGroup)

  const textObjects = []
  const wallMeshes = []

  const config = {
    wallHeight: 50,
    wallColor: '#888888',
    textScale: 0.5,
    textDepth: 5,
    textOpacity: 1.0,
    textColor: 0xff0000,
    textXOffset: 0,
    textYOffset: 0,
    textZPosition: 10,
    globalScale: 0.1
  }

  const gui = new dat.GUI()
  // 墙体控制
  const wallFolder = gui.addFolder('墙体设置')
  wallFolder.add(config, 'wallHeight', 10, 200).step(1).name('墙体高度').onChange(updateWallHeight)
  wallFolder.addColor(config, 'wallColor').name('墙体颜色').onChange(updateWallColor)

  // 文本控制
  const textFolder = gui.addFolder('文本设置')
  textFolder.add(config, 'textScale', 0.01, 5.0).step(0.01).name('文本缩放')
  textFolder.add(config, 'textDepth', 1, 20).step(1).name('文本深度')
  textFolder.add(config, 'textOpacity', 0.1, 1.0).step(0.1).name('文本透明度')
  textFolder.addColor(config, 'textColor').name('文本颜色')
  textFolder.add(config, 'textXOffset', -100, 100).step(1).name('X轴偏移')
  textFolder.add(config, 'textYOffset', -100, 100).step(1).name('Y轴偏移')
  textFolder.add(config, 'textZPosition', 0, 100).step(1).name('Z轴位置')

  function updateWallHeight() {
    wallMeshes.forEach(mesh => {
      mesh.geometry.dispose()
      const shape = mesh.userData.shape
      mesh.geometry = new THREE.ExtrudeGeometry(shape, {
        depth: config.wallHeight,
        bevelEnabled: false
      })
    })
  }

  function updateWallColor() {
    wallMeshes.forEach(mesh => {
      mesh.material.color.setStyle(config.wallColor)
    })
  }

  function updateTextProperties() {
    textObjects.forEach(text => {
      text.scale.set(config.textScale, config.textScale, config.textScale)
      text.depth = config.textDepth
      text.material.opacity = config.textOpacity
      text.material.transparent = config.textOpacity < 1.0
      text.color = config.textColor

      const originalPosition = text.userData.originalPosition
      text.position.set(
        originalPosition.x + config.textXOffset,
        originalPosition.y + config.textYOffset,
        config.textZPosition
      )
    })
  }

  async function loadSVG() {
    try {
      const response = await fetch('/floorplan.svg')
      const svgString = await response.text()

      const parser = new DOMParser()
      const svgDoc = parser.parseFromString(svgString, 'image/svg+xml')
      const svgRoot = svgDoc.querySelector('svg')

      if (!svgRoot) {
        console.error('SVG解析失败')
        return
      }

      const viewBox = svgRoot.getAttribute('viewBox')
      let viewBoxX = 0, viewBoxY = 0, viewBoxWidth = 1000, viewBoxHeight = 1000
      if (viewBox) {
        const parts = viewBox.split(' ').map(parseFloat)
        if (parts.length === 4) {
          [viewBoxX, viewBoxY, viewBoxWidth, viewBoxHeight] = parts
        }
      }

      const offsetX = -viewBoxWidth / 2 - viewBoxX
      const offsetY = viewBoxHeight / 2 + viewBoxY

      const loader = new SVGLoader()
      const svgData = loader.parse(svgString)

      svgData.paths.forEach((path) => {
        const shapes = SVGLoader.createShapes(path)
        shapes.forEach((shape) => {
          const geometry = new THREE.ExtrudeGeometry(shape, {
            depth: config.wallHeight,
            bevelEnabled: false
          })

          let color = path.userData.style.fill || config.wallColor
          if (color === 'none') color = config.wallColor

          const material = new THREE.MeshPhongMaterial({
            color: new THREE.Color().setStyle(color),
            side: THREE.DoubleSide
          })
          const mesh = new THREE.Mesh(geometry, material)
          mesh.userData.shape = shape // 保存形状以便更新高度

          // 坐标修正 + 缩放
          mesh.scale.set(config.globalScale, config.globalScale, config.globalScale)
          mesh.position.set(offsetX * config.globalScale, -offsetY * config.globalScale, 0)

          objectGroup.add(mesh)
          wallMeshes.push(mesh)
        })
      })

      const textNodes = svgDoc.getElementsByTagName('text')
      Array.from(textNodes).forEach((textNode) => {
        const textContent = textNode.textContent.trim()
        if (!textContent) return

        let x = parseFloat(textNode.getAttribute('x') || 0)
        let y = parseFloat(textNode.getAttribute('y') || 0)

        const transform = textNode.getAttribute('transform')
        if (transform) {
          if (transform.includes('matrix')) {
            const matrixValues = transform.replace(/matrix\(|\)/g, '').split(/[\s,]+/).map(Number)
            if (matrixValues.length === 6) {
              x = matrixValues[4]
              y = matrixValues[5]
            }
          } else if (transform.includes('translate')) {
            const translateValues = transform.replace(/translate\(|\)/g, '').split(/[\s,]+/).map(Number)
            if (translateValues.length >= 2) {
              x += translateValues[0]
              y += translateValues[1]
            }
          }
        }

        const fontSize = parseFloat(textNode.getAttribute('font-size') || 12)
        const textAnchor = textNode.getAttribute('text-anchor') || 'start'
        const dominantBaseline = textNode.getAttribute('dominant-baseline') || 'alphabetic'

        const text = new Text()
        text.text = textContent
        text.fontSize = fontSize * 10 * config.globalScale
        text.color = config.textColor
        text.fontFamily = 'sans-serif'

        switch (textAnchor) {
          case 'middle': text.anchorX = 'center'; break
          case 'end': text.anchorX = 'end'; break
          default: text.anchorX = 'start'
        }

        switch (dominantBaseline) {
          case 'middle': text.anchorY = 'middle'; break
          case 'hanging': text.anchorY = 'top'; break
          case 'text-bottom': text.anchorY = 'bottom'; break
          default: text.anchorY = 'alphabetic'
        }

        const finalX = (x + offsetX) * config.globalScale
        const finalY = (-y + offsetY) * config.globalScale

        text.position.set(finalX, finalY, config.textZPosition)
        text.userData.originalPosition = new THREE.Vector3(finalX, finalY, config.textZPosition)
        text.scale.set(config.textScale, config.textScale, config.textScale)

        textGroup.add(text)
        textObjects.push(text)
      })

      const box = new THREE.Box3().setFromObject(objectGroup)
      const center = box.getCenter(new THREE.Vector3())
      const size = box.getSize(new THREE.Vector3())

      const maxDim = Math.max(size.x, size.y, size.z)
      const fov = camera.fov * (Math.PI / 180)
      const cameraZ = Math.max(maxDim / (2 * Math.tan(fov / 2)), 500)

      camera.position.set(center.x, center.y, cameraZ)
      camera.lookAt(center)
      controls.target.copy(center)
    } catch (error) {
      console.error('加载SVG失败:', error)
    }
  }

  loadSVG()

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })

  gui.__controllers.forEach(controller => {
    if (controller.property !== 'globalScale') {
      controller.onChange(updateTextProperties)
    }
  })

  const animate = () => {
    requestAnimationFrame(animate)
    controls.update()
    textObjects.forEach(text => text.sync())
    renderer.render(scene, camera)
  }
  animate()
})
</script>

<style>
.scene-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  position: fixed;
  top: 0;
  left: 0;
}
.dg.ac {
  z-index: 100 !important;
}
</style>