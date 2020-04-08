<template>
    <canvas ref="canvas"
            class="points-cloud">
    </canvas>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';
import * as THREE from 'three';
import { Interaction } from 'three.interaction';

@Component
export default class PointsCloudComponent extends Vue {
    @Prop({ default: 500 })
    width!: number;

    @Prop({ default: 500 })
    height!: number;

    @Prop({ default: () => { return { items: [] }}})
    data!: {
        items: {
            title: string;
            link: string;
            filled: boolean;
        }[];
    };

    viewAngle = 45;
    near = 0.1;
    far = 1800;
    renderer!: THREE.WebGLRenderer;
    camera!: THREE.PerspectiveCamera;
    scene!: THREE.Scene;
    points!: THREE.Group;
    sphere!: THREE.Mesh;
    allObjects!: THREE.Group;
    needToRotate = true;

    get aspect() {
        return this.width / this.height;
    }

    mounted() {
        this.init();
        this.animate();
    }

    async init() {
        this.allObjects = new THREE.Group()
        //renderer
        this.renderer = new THREE.WebGLRenderer({
            canvas: this.$refs.canvas as HTMLCanvasElement,
            antialias: true
        });
        this.renderer.setSize(this.width, this.height);
        this.renderer.setPixelRatio(window.devicePixelRatio);

        //camera
        this.camera = new THREE.PerspectiveCamera(this.viewAngle, this.aspect, this.near, this.far);
        this.camera.position.z = 1000;

        //lights
        this.renderer.outputEncoding = THREE.sRGBEncoding;

        this.renderer.setClearColor(0xffffff, 1.0);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);

        directionalLight.position.set(0, -1, 0);

        this.scene = new THREE.Scene();
        this.scene.add(this.camera);
        this.scene.add(directionalLight);

        new Interaction(this.renderer, this.scene, this.camera);

        this.points = new THREE.Group();

        const distance = 300;

        this.data.items.forEach((item) => {
            const theta = THREE.MathUtils.randFloatSpread(360);
            const phi = THREE.MathUtils.randFloatSpread(360);
            const circle = this.drawCircle({ fill: item.filled });

            circle.position.x = distance * Math.sin(theta) * Math.cos(phi);
            circle.position.y = distance * Math.sin(theta) * Math.sin(phi);
            circle.position.z = distance * Math.cos(theta);

            this.points.add(circle);
        });

        const sphereGeometry = new THREE.SphereGeometry(distance, 24, 24);
        const sphereMaterial = new THREE.MeshBasicMaterial({
            //color: 0x00ff00,
            //transparent: true,
            //opacity: 0,
            visible: false
        });

        this.sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);

        this.allObjects.add(this.points)
        this.allObjects.add(this.sphere)

        this.allObjects.on('mouseover', () => {
            this.needToRotate = false;
        })
        this.allObjects.on('mouseout', () => {
            this.needToRotate = true;
        })

        this.scene.add(this.allObjects);
    }

    drawCircle(options?: { size?: number; borderWidth?: number; fill?: boolean }) {
        const size = options?.size ?? 8;
        const borderWidth = options?.borderWidth ?? 1;
        const fill = options?.fill ?? false;
        const canvas = document.createElement('canvas');
        const virtualBorderWidth = borderWidth * 2;
        const virtualSize = size * 2;

        canvas.width = virtualSize + 1;
        canvas.height = virtualSize + 1;

        const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;

        ctx.strokeStyle = `rgba(0, 0, 0, 1)`;
        ctx.lineWidth = virtualBorderWidth;

        ctx.beginPath();
        ctx.arc(virtualSize / 2, virtualSize / 2, (virtualSize - virtualBorderWidth) / 2, 0, Math.PI * 2);
        ctx.closePath()
        ctx.stroke();
        if (fill) {
            ctx.fill()
        }

        const texture = new THREE.CanvasTexture(canvas);
        const spriteMaterial = new THREE.SpriteMaterial({
            map: texture,
            color: 0xffffff,
            fog: true
        })
        const sprite = new THREE.Sprite(spriteMaterial);

        sprite.scale.set(size, size, 1);

        return sprite;
    }

    animate() {
        window.requestAnimationFrame(this.animate);
        if (this.needToRotate) {
            this.points.rotation.y += 0.005;
        }
        this.renderer.render(this.scene, this.camera);
    }
}
</script>

<style lang="stylus">
.points-cloud {
    background: yellow;

}
canvas {
    box-shadow: 0 0 3px 0px #00ff41;
}
</style>
