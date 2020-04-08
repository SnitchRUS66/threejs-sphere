<template>
    <div ref="pointsCloud"
         class="points-cloud">
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import * as THREE from 'three';

@Component
export default class PointsCloudComponent extends Vue {
    width = 0;
    height = 0;
    viewAngle = 45;
    near = 0.1;
    far = 10000;
    renderer!: THREE.WebGLRenderer;
    camera!: THREE.PerspectiveCamera;
    scene!: THREE.Scene;
    points!: THREE.Points;

    get aspect() {
        return this.width / this.height;
    }

    mounted() {
        this.init();
        this.animate();
    }

    init() {
        const container = this.$refs.pointsCloud as HTMLDivElement;

        this.width = container.clientWidth;
        this.height = container.clientHeight;

        //renderer
        this.renderer = new THREE.WebGLRenderer();
        this.renderer.setSize(this.width, this.height);
        container.appendChild(this.renderer.domElement);

        //camera
        this.camera = new THREE.PerspectiveCamera(this.viewAngle, this.aspect, this.near, this.far);
        this.camera.position.z = 300;

        //lights
        //this.renderer.gammaInput = true;
        //this.renderer.gammaOutput = true;
        this.renderer.outputEncoding = THREE.sRGBEncoding;

        this.renderer.setClearColor(0x909090, 1.0);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);

        directionalLight.position.set(0, -1, 0);

        this.scene = new THREE.Scene();
        this.scene.add(this.camera);
        this.scene.add(directionalLight);

        const distance = 100;
        const geometry = new THREE.Geometry();

        for (let i = 0; i < 1000; i++) {

            const vertex = new THREE.Vector3();

            const theta = THREE.MathUtils.randFloatSpread(360);
            const phi = THREE.MathUtils.randFloatSpread(360);

            vertex.x = distance * Math.sin(theta) * Math.cos(phi);
            vertex.y = distance * Math.sin(theta) * Math.sin(phi);
            vertex.z = distance * Math.cos(theta);

            geometry.vertices.push(vertex);
        }
        geometry.computeBoundingSphere();
        this.points = new THREE.Points(geometry, new THREE.PointsMaterial({
            color: 0xffffff,
            size: 5
        }));
        //this.points.boundingSphere = 5;
        this.scene.add(this.points);
    }

    animate() {
        window.requestAnimationFrame(this.animate);
        this.points.rotation.y += 0.001;
        this.renderer.render(this.scene, this.camera);
    }
}
</script>

<style lang="stylus">
.points-cloud {
    width: 400px;
    height: 400px;
    background #dfbc1f;

    canvas {
        width: 100% !important;
        height: 100% !important;
    }
}
</style>
