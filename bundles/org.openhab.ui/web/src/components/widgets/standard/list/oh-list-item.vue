<template>
  <f7-list-button v-if="config.listButton && !context.editmode" :title="config.title || 'Action'" :color="config.color || 'blue'" @click="performAction" />
  <f7-list-item divider :title="config.title" v-else-if="config.divider && !context.editmode" />
  <f7-list-item v-else v-bind="config" :divider="config.divider && !context.editmode"
                :media-item="context.parent.component.config.mediaList && !config.divider"
                :badge="(config.divider) ? 'Divider' : (config.listButton) ? 'List button' : config.badge"
                :accordion-item="context.component.slots && context.component.slots.accordion && !config.divider && !context.editmode"
                :link="(config.action !== undefined && config.action !== '' && !context.editmode) ? true : undefined"
                @click="performAction">
    <slot name="inner" #inner />
    <slot name="after" #after />
    <slot name="content" #content />
    <slot name="root-end" #root-end />
    <slot name="footer" #footer />
    <generic-widget-component slot="after" v-if="context.component.slots && context.component.slots.after && context.component.slots.after.length"
                              :context="childContext(context.component.slots.after[0])" v-on="$listeners" />
    <f7-accordion-content v-if="context.parent.component.config.accordionList && !context.editmode">
      <generic-widget-component v-if="context.component.slots && context.component.slots.accordion && context.component.slots.accordion.length"
                                :context="childContext(context.component.slots.accordion[0])" v-on="$listeners" />
      <!-- <oh-placeholder-widget v-else-if="context.editmode" class="oh-column-item placeholder" @click="context.editmode.addWidget(context.component, null, context.parent, 'accordion')" /> -->
    </f7-accordion-content>
    <oh-icon slot="media" v-if="config.icon" :icon="config.icon" height="32" width="32" :color="config.iconColor" :state="(config.item && config.iconUseState) ? context.store[config.item].state : null" />
    <span slot="media" v-else-if="config.fallbackIconToInitial && config.title && context.parent.component.config && context.parent.component.config.mediaList" class="item-initial">{{ config.title[0].toUpperCase() }}</span>
  </f7-list-item>
</template>

<script>
import mixin from '../../widget-mixin'
import { actionsMixin } from '../../widget-actions'
import { OhListItemDefinition } from '@/assets/definitions/widgets/standard/listitems'

export default {
  name: 'oh-list-item',
  mixins: [mixin, actionsMixin],
  widget: OhListItemDefinition
}
</script>
