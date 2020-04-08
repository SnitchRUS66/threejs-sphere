<template>
    <div class="home">
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
        margin: 20px auto 0 auto;
    }
}
</style>
