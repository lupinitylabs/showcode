<template>
    <Interact
        :style="{
            touchAction: 'none',
            width: `${width}px`,
            height: `${height}px`,
        }"
        :resizable="resizable"
        @resizemove="onResize"
    >
        <div class="absolute inset-0 z-[2] w-full h-full bg-overlay pointer-events-none"></div>

        <div
            v-bind="backgroundAttributes"
            class="absolute inset-0"
            :dusk="`background-${background}`"
            :data-hide="background === 'transparent'"
        ></div>

        <!-- Optional grid. Left out for a future implementation. -->
        <!-- <div class="absolute z-[2] w-full h-full bg-grid pointer-events-none"></div> -->

        <ButtonResize
            ref="top"
            data-hide
            class="absolute top-0 left-1/2 -m-1.5 cursor-resize-height resize-top"
        />

        <ButtonResize
            ref="bottom"
            data-hide
            class="absolute bottom-0 left-1/2 -m-1.5 cursor-resize-height resize-bottom"
        />

        <ButtonResize
            ref="left"
            data-hide
            class="absolute left-0 top-1/2 -m-1.5 cursor-resize-width resize-left"
        />

        <ButtonResize
            data-hide
            ref="right"
            class="absolute right-0 top-1/2 -m-1.5 cursor-resize-width resize-right"
        />

        <div class="relative flex items-center justify-center flex-1">
            <slot />

            <Divider
                data-hide
                :number="height"
                class="absolute top-0 right-0 mx-4 text-xs font-semibold -mr-14 text-ui-gray-300"
            />
        </div>

        <Separator
            :number="width"
            class="absolute bottom-0 w-full text-xs font-semibold -mb-14 text-ui-gray-300"
        />
    </Interact>
</template>

<script>
import interact from 'interactjs';
import { ref, toRefs, computed } from '@nuxtjs/composition-api';

export default {
    props: {
        width: {
            type: Number,
            required: true,
        },
        height: {
            type: Number,
            required: true,
        },
        aspectRatio: {
            type: Array,
            required: false,
        },
        background: {
            type: String,
            required: true,
        },
        backgroundAttributes: {
            type: Object,
            required: true,
        },
    },

    setup(props, { emit }) {
        const x = ref(null);
        const y = ref(null);

        const top = ref(null);
        const right = ref(null);
        const bottom = ref(null);
        const left = ref(null);

        const { aspectRatio } = toRefs(props);

        const ratio = computed(() => {
            if (aspectRatio.value) {
                const [ratioX, ratioY] = aspectRatio.value;

                return ratioX / ratioY;
            }
        });

        const resizable = computed(() => ({
            edges: {
                top: top.value?.$el,
                right: right.value?.$el,
                bottom: bottom.value?.$el,
                left: left.value?.$el,
            },
            modifiers: [
                interact.modifiers.aspectRatio({
                    ratio: ratio.value,
                }),
            ],
        }));

        const onResize = (event) => {
            const container = event.target.parentNode;

            if (event.rect.width) {
                const scaleX = container.getBoundingClientRect().width / container.offsetWidth;

                const x = event.rect.width / scaleX;

                emit('update:width', x);
            }

            if (event.rect.height) {
                const scaleY = container.getBoundingClientRect().height / container.offsetHeight;

                const y = event.rect.height / scaleY;

                emit('update:height', y);
            }
        };

        return { x, y, top, right, bottom, left, resizable, onResize };
    },
};
</script>
