<template>
    <div class="home">
        <points-cloud :data="testData" />
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
    .points-cloud {
        margin: 0 auto;
    }
}
</style>
