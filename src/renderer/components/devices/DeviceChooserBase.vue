<template>
  <div>
    <div v-if="selected">
      <v-card-text class="subheading pb-0 mb-0">{{ chosenText }}</v-card-text>
      <v-list two-line dense>
        <v-list-tile avatar :key="selected.token">
          <v-list-tile-avatar>
            <img :src="selected.deviceType.imageUrl" />
          </v-list-tile-avatar>
          <v-list-tile-content>
            <v-list-tile-title v-html="selected.deviceType.name + ' (' + selected.token + ')'"></v-list-tile-title>
            <v-list-tile-sub-title v-html="selected.deviceType.description"></v-list-tile-sub-title>
          </v-list-tile-content>
          <v-list-tile-action>
            <v-btn icon ripple @click.stop="onRemoved(true)">
              <v-icon class="grey--text">delete</v-icon>
            </v-btn>
          </v-list-tile-action>
        </v-list-tile>
      </v-list>
    </div>
    <div v-else>
      <v-card-text class="subheading pb-0 mb-0">{{ notChosenText }}</v-card-text>
      <v-list v-if="all" class="scrolling-list" two-line dense>
        <template v-for="item in all">
          <v-list-tile avatar :key="item.token" @click="onSelected(item, true)">
            <v-list-tile-avatar>
              <img :src="item.deviceType.imageUrl" />
            </v-list-tile-avatar>
            <v-list-tile-content>
              <v-list-tile-title v-html="item.deviceType.name + ' (' + item.token + ')'"></v-list-tile-title>
              <v-list-tile-sub-title v-html="item.deviceType.description"></v-list-tile-sub-title>
            </v-list-tile-content>
          </v-list-tile>
        </template>
      </v-list>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Watch } from "sitewhere-ide-common";
import Vue from "vue";

import { IDevice, IDeviceType } from "sitewhere-rest-api";

@Component({})
export default class DeviceChooserBase extends Vue {
  @Prop() readonly value!: string;
  @Prop() readonly chosenText!: string;
  @Prop() readonly notChosenText!: string;
  @Prop() readonly all!: IDevice[];

  selected: IDevice | null = null;

  get deviceToken(): string | null {
    return this.selected ? this.selected.token : null;
  }

  get deviceType(): IDeviceType {
    return this.selected ? (this.selected as any).deviceType : null;
  }

  get deviceTypeName() {
    return this.deviceType ? this.deviceType.name : null;
  }

  get deviceTypeDesc() {
    return this.deviceType ? this.deviceType.description : null;
  }

  @Watch("value", { immediate: true })
  onValueChanged(updated: string) {
    this.onValueUpdated(updated);
  }

  /** Called to update choice based on token */
  onValueUpdated(token: string) {
    let found: boolean = false;
    this.all.forEach(item => {
      if (item.token === token) {
        found = true;
        this.onSelected(item, false);
      }
    });
    if (!found) {
      this.onRemoved(false);
    }
  }

  /** Called to select an item */
  onSelected(entity: IDevice, emit: boolean) {
    this.selected = entity;
    if (emit) {
      this.$emit("selected", entity.token);
    }
  }

  /** Called to remove selection */
  onRemoved(emit: boolean) {
    this.selected = null;
    if (emit) {
      this.$emit("selected", null);
    }
  }
}
</script>

<style scoped>
.scrolling-list {
  max-height: 300px;
  overflow-y: auto;
}
</style>
