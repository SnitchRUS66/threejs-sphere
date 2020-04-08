<template>
    <div class="home">
        <h1>This is an home page</h1>
        <points-cloud />
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import PointsCloudComponent from '@/components/PointsCloud.vue';

@Component({
    components: {
        'points-cloud': PointsCloudComponent
    }
})
export default class HomeView extends Vue {}
</script>

<style lang="stylus">
.home {
    .points-cloud {
        margin: 0 auto;
    }
}
</style>
