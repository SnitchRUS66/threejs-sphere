<template>
    <div class="points-cloud">
        <div class="points-cloud__labels-list">
            <a v-for="(item, index) in data.items.slice(0, maxLabels)"
               ref="label"
               class="points-cloud__label"
               :key="`${item.title}_${index}`"
               :href="item.link"
               target="_blank"
               @mouseover="onMouseoverLabel"
               @mouseout="onMouseoutLabel">
                {{item.title}}
            </a>
        </div>
        <canvas ref="canvas"
                class="points-cloud__canvas">
        </canvas>
    </div>
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

    @Prop({ default: 300 })
    maxPoints!: number;

    @Prop({ default: 20 })
    maxLabels!: number;

    viewAngle = 45;
    near = 0.1;
    far = 1800;
    renderer!: THREE.WebGLRenderer;
    camera!: THREE.PerspectiveCamera;
    scene!: THREE.Scene;
    points!: THREE.Group;
    sphere!: THREE.Mesh;
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    allObjects!: THREE.Group & { on?: (event: string, cb: () => any) => void};
    needToRotate = true;
    tempVector = new THREE.Vector3();

    get aspect() {
        return this.width / this.height;
    }

    mounted() {
        this.init();
        this.animate();
    }

    init() {
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

        this.allObjects = new THREE.Group();
        this.points = new THREE.Group();

        const distance = 300;

        this.data.items.some((item, index) => {
            if (index >= this.maxPoints) {
                return true;
            }

            const theta = THREE.MathUtils.randFloatSpread(360);
            const phi = THREE.MathUtils.randFloatSpread(360);
            const circleParams = {
                fill: item.filled,
                color: index < this.maxLabels
                    ? `rgba(255, 0, 0, 1)`
                    : `rgba(0, 0, 0, 1)`
            }
            const circle = this.drawCircle(circleParams);

            circle.position.x = distance * Math.sin(theta) * Math.cos(phi);
            circle.position.y = distance * Math.sin(theta) * Math.sin(phi);
            circle.position.z = distance * Math.cos(theta);

            this.points.add(circle);
        });

        const sphereGeometry = new THREE.SphereGeometry(distance, 24, 24);
        const sphereMaterial = new THREE.MeshBasicMaterial({
            visible: false
        });

        this.sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);

        this.allObjects.add(this.points);
        this.allObjects.add(this.sphere);

        /* eslint-disable */
        if (this.allObjects.on) {
            this.allObjects.on('mouseover', () => {
                this.needToRotate = false;
                (this.$refs.canvas as HTMLCanvasElement).style.cursor = 'not-allowed';
            })
            this.allObjects.on('mouseout', () => {
                this.needToRotate = true;
                (this.$refs.canvas as HTMLCanvasElement).style.cursor = 'auto';
            })
        }
        /* eslint-enable */

        this.scene.add(this.allObjects);
    }

    onMouseoverLabel() {
        this.needToRotate = false;
    }

    onMouseoutLabel() {
        this.needToRotate = true;
    }

    drawCircle(
        options?: {
            size?: number;
            borderWidth?: number;
            fill?: boolean;
            color?: string | CanvasGradient | CanvasPattern;
        }
    ) {
        const size = options?.size ?? 8;
        const borderWidth = options?.borderWidth ?? 1;
        const fill = options?.fill ?? false;
        const color = options?.color ?? `rgba(0, 0, 0, 1)`;
        const canvas = document.createElement('canvas');
        const virtualBorderWidth = borderWidth * 2;
        const virtualSize = size * 2;

        canvas.width = virtualSize + 1;
        canvas.height = virtualSize + 1;

        const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;

        ctx.strokeStyle = color;
        ctx.fillStyle = color;
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

    updateLabelsPoosition() {
        this.points.children.forEach((point, index) => {
            if (index < this.maxLabels) {
                point.getWorldPosition(this.tempVector)
                this.tempVector.project(this.camera);

                const x = (this.tempVector.x * 0.5 + 0.5) * this.width;
                const y = (this.tempVector.y * -0.5 + 0.5) * this.height;
                const label = (this.$refs.label as HTMLDivElement[])[index];

                if (label) {
                    label.style.transform = `translate(-50%, -50%) translate(${x}px,${y}px)`;
                    //label.style.zIndex = `${(-this.tempVector.z * .5 + .5) * 100000 | 0}`;
                }
            }
        })
    }

    animate() {
        window.requestAnimationFrame(this.animate);
        if (this.needToRotate) {
            this.points.rotation.y += 0.005;
            this.renderer.render(this.scene, this.camera);
            this.updateLabelsPoosition();
        }
    }
}
</script>

<style lang="stylus">
.points-cloud {
    display: table;
    &__labels-list {
        position: relative;
    }
    &__label {
        position: absolute;
        display: flex;
        align-items: center;
        background: #fff;
        height: 24px;
        padding: 0 6px;
        font-size: 13px;
        color: #000;
        text-decoration: none;
        border: 1px solid #000;
        border-radius: 12px;
        cursor: pointer;
        user-select: none;
    }
}
</style>
