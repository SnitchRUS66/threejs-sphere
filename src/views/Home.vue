<template>
    <div class="home">
        <points-cloud :data="testData"
                      :max-points="300"
                      :max-labels="9" />
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import PointsCloudComponent from '@/components/PointsCloud.vue';
import testData from '@/data.json';

@Component({
    components: {
        'points-cloud': PointsCloudComponent
    }
})
export default class HomeView extends Vue {
    testData = testData as {
        items: {
            title: string;
            link: string;
            filled: boolean;
        }[];
    };
}
</script>

<style lang="stylus">
.home {
    height: 100%;
    display: flex;
    align-items: center;
    .points-cloud {
        margin: 0 auto;
    }
}
</style>
