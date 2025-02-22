<template>
    <v-expansion-panel>
        <v-expansion-panel-header>
            <div class="unit-header" style="display: flex; align-items: center; gap: 10px">
                <component :is="iconType" v-if="showUnitIcons" id="iconType" class="unit-icon" />
                <div v-else style="padding-left: 10px" />
                <h2 class="unit-title" style="margin: 0">{{ formattedUnitName }}</h2>
                <span v-if="unit.hubs.length > 0" class="hub-container">
                    <strong>{{ $t('Panels.AfcPanel.Hub') }}</strong>
                    <v-tooltip top>
                        <template #activator="{ on, attrs }">
                            <span
                                v-bind="attrs"
                                :class="{
                                    'status-light': true,
                                    success: currentHubState,
                                    error: !currentHubState,
                                }"
                                v-on="on"></span>
                        </template>
                        <span>{{ hubStatus }}</span>
                    </v-tooltip>
                </span>
            </div>
        </v-expansion-panel-header>
        <v-expansion-panel-content>
            <div class="lanes-container">
                <afc-units-item-lane
                    v-for="(lane, index) in unit.lanes"
                    :key="index"
                    :lane="lane"
                    class="lane-card rounded-lg shadow-md overflow-hidden" />
            </div>
        </v-expansion-panel-content>
    </v-expansion-panel>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins, Prop } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import AfcUnitsItemLane from '@/components/panels/Afc/AfcUnitsItemLane.vue'
import BoxTurtleIcon from '@/components/ui/BoxTurtleIcon.vue'
import NightOwlIcon from '@/components/ui/NightOwlIcon.vue'
import { Unit, Lane, Hub } from '@/store/server/afc/types'

@Component({
    components: { AfcUnitsItemLane, BoxTurtleIcon, NightOwlIcon },
})
export default class AfcUnitsItem extends Mixins(BaseMixin) {
    @Prop({ type: Object, required: true }) readonly unit!: Unit

    currentHubState = false

    get formattedUnitName(): string {
        return String(this.unit.name).replace(/_/g, ' ')
    }

    get showUnitIcons(): boolean {
        return this.$store.state.gui.view.afc.showUnitIcons ?? true
    }

    get currentLane(): Lane {
        return this.$store.getters['server/afc/getCurrentLane']
    }

    get currentLoad(): Lane {
        return this.$store.getters['server/afc/getCurrentLoad']
    }

    get hubStatus(): string {
        if (this.currentLoad && this.unit.hubs.some((hub) => hub === this.currentLoad.hub)) {
            this.hubStatusBool(this.currentLoad.hub)
            return this.hubStatusString(this.currentLoad.hub)
        } else if (this.currentLane && this.unit.hubs.some((hub) => hub === this.currentLane.hub)) {
            this.hubStatusBool(this.currentLane.hub)
            return this.hubStatusString(this.currentLane.hub)
        } else {
            this.hubStatusBool(this.unit.hubs[0])
            return this.hubStatusString(this.unit.hubs[0])
        }
    }

    hubStatusString(hub: Hub): string {
        const status = hub.state ? this.$t('Panels.AfcPanel.Detected') : this.$t('Panels.AfcPanel.Empty')
        return `${this.$t('Panels.AfcPanel.HubStatus', { unit: this.formattedUnitName })} - ${status}`
    }

    hubStatusBool(hub: Hub) {
        this.currentHubState = hub.state
    }

    get iconType() {
        switch (this.unit.type) {
            case 'Box_Turtle':
                return BoxTurtleIcon
            case 'Night_Owl':
                return NightOwlIcon
            default:
                return null
        }
    }
}
</script>

<style scoped>
.lanes-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 8px;
    padding-top: 0px;
}

.hub-container {
    text-align: left;
    margin: 20px 0;
    padding-left: 10px;
}

.hub-container strong {
    margin-right: 5px;
}

.unit-title {
    font-size: 1.5em;
    margin-bottom: 16px;
    text-align: left;
}

.lane-card {
    transition: background-color 0.3s ease;
    padding-top: 0px;
    max-width: 150px;
    min-width: 110px;
    flex: 1 1 110px;
    position: relative;
    text-align: right;
    width: 100%;
}

.theme--dark .lane-card {
    background-color: #2e2e2e;
}

.theme--light .lane-card {
    background-color: #ebebeb;
}

.status-light {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    display: inline-block;
    margin-right: 5px;
    margin-left: 5px;
}

.unit-icon {
    width: 70px;
    height: 70px;
    margin-left: 10px;
}

.lane-status {
    margin-right: 10px;
    margin-left: 5px;
}
</style>
