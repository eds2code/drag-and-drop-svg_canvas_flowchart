<template>
    <v-container>
        <v-row>
            <v-col cols="12">
                <SvgCanvas />
            </v-col>
        </v-row>
    </v-container>
</template>

<script lang="ts">
import {Vue, Component} from 'vue-property-decorator'
import SvgCanvas from '@/components/SvgCanvas.vue'

@Component({
    components: {
        SvgCanvas
    }
})
export default class Home extends Vue {
    
}
</script>