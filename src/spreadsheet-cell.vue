<template>
    <div ref="target" tabindex="0" class="spreadsheet-cell" :class="{ 'edit-mode': editMode, [interfaceId]: true }"
        @dblclick="enterCell">
        <slot v-if="editMode" name="interface" />
        <slot v-else name="display" :display-item="mergedItemWithEdits" />

        <div v-if="!editMode && fieldHasEdits" v-tooltip="t('unsaved_changes')" class="edit-marker"></div>
    </div>
</template>



<script setup lang="ts">
    import { nextTick, ref, computed, type Ref } from 'vue'
    import { useI18n } from 'vue-i18n';
    import { onKeyStroke, onClickOutside } from '@vueuse/core'

    const props = defineProps<{
        column: number;
        item: any;
        fieldKey: string;
        fieldEdits?: any;
        interfaceId: string;
    }>();

    const emit = defineEmits(['leaveCell']);

    const target = ref(null);

    const { editMode, enterCell } = useEditMode({ onFocusCell });

    useCellNavigation(editMode);

    const { t } = useI18n();
    const { fieldHasEdits, mergedItemWithEdits } = useDisplayEdits();

    function onFocusCell() {
        // CLICK

        if (['select-dropdown', 'datetime', 'select-multiple-dropdown'].includes(props.interfaceId)) {
            ((target.value! as HTMLElement)?.querySelector('.v-input') as HTMLElement)?.click();
            return;
        }

        // CLICK AFTER DELAY

        if (['collection-item-dropdown', 'file', 'select-dropdown-m2o'].includes(props.interfaceId)) {
            setTimeout(() => {
                ((target.value! as HTMLElement)?.querySelector('.v-input') as HTMLElement)?.click();
            }, 50);
            return;
        }

        // FOCUS

        if (props.interfaceId == 'boolean') {
            ((target.value! as HTMLElement)?.querySelector('button.v-checkbox') as HTMLElement)?.focus();
            return;
        }

        if (['input-autocomplete-api', 'select-icon', 'input-hash'].includes(props.interfaceId)) {
            ((target.value! as HTMLElement)?.querySelector('.v-input input') as HTMLElement)?.focus();
            return;
        }

        if (['select-color'].includes(props.interfaceId)) {
            ((target.value! as HTMLElement)?.querySelector('.v-input.color-input > .input > input') as HTMLElement)?.focus();
            return;
        }

        ((target.value! as HTMLElement)?.querySelector('.v-input') as HTMLElement)?.focus();
    }

    function useEditMode({ onFocusCell }: any) {
        const editMode = ref(false);

        // `{ eventName: 'keyup' }` is important to prevent focussed fields like `v-checkbox` to be triggered while entering the cell
        onKeyStroke('Enter', enterCell, { target, eventName: 'keyup' });
        onKeyStroke(['Esc', 'Escape'], leaveCellAndFocus);
        onKeyStroke('Tab', leaveCell, { target });
        onClickOutside(target, clickOutside);

        return { editMode, enterCell };

        function enterCell() {
            if (editMode.value) return;
            editMode.value = true;
            nextTick(onFocusCell);
        }

        function leaveCell() {
            if (!editMode.value) return;
            editMode.value = false;
            emit('leaveCell');
        }

        function leaveCellAndFocus() {
            if (!editMode.value) return;
            leaveCell();
            (target.value! as HTMLElement)?.focus();
        }

        function clickOutside(e: MouseEvent) {
            if (!editMode.value) return;
            const elementsToIgnore = (e.target as HTMLElement)?.closest('#dialog-outlet, #menu-outlet');
            if (elementsToIgnore) return;
            leaveCell();
        }
    }

    function useCellNavigation(preventNavigation: Ref<boolean>) {
        onKeyStroke(['Left', 'ArrowLeft'], (e) => cellNavigation(e, {}), { target });
        onKeyStroke(['Right', 'ArrowRight'], (e) => cellNavigation(e, { next: true }), { target });
        onKeyStroke(['Up', 'ArrowUp'], (e) => cellNavigation(e, { vertical: true }), { target });
        onKeyStroke(['Down', 'ArrowDown'], (e) => cellNavigation(e, { vertical: true, next: true }), { target });

        function cellNavigation(e: KeyboardEvent, { vertical = false, next = false }) {
            if (preventNavigation.value) return;

            e.preventDefault();

            const parent = vertical ? 'tr.table-row' : 'td.cell';
            const parentSibling = (e.target as HTMLElement)?.closest(parent)?.[next ? 'nextElementSibling' : 'previousElementSibling'];
            const cell = vertical
                ? parentSibling?.querySelectorAll('.spreadsheet-cell')?.[props.column]
                : parentSibling?.querySelector('.spreadsheet-cell');

            (cell as HTMLElement)?.focus()
        }
    }

    function useDisplayEdits() {
        const fieldHasEdits = computed(() => typeof props.fieldEdits !== 'undefined');

        const mergedItemWithEdits = computed(() => {
            if (!fieldHasEdits.value) return props.item;

            return {
                ...props.item,
                [props.fieldKey]: props.fieldEdits
            };
        });

        return { fieldHasEdits, mergedItemWithEdits };
    }
</script>



<style lang="scss" scoped>
    .spreadsheet-cell {
        display: flex;
        align-items: center;
        position: relative;
        width: 100%;
        height: 100%;

        &:not(.edit-mode) {
            overflow: hidden;
            text-align: inherit;
            justify-content: inherit;
            white-space: nowrap;
            text-overflow: ellipsis;
            border: var(--theme--border-width) solid transparent;
            border-radius: var(--v-input-border-radius, var(--theme--border-radius));

            &:hover {
                border-color: var(--theme--border-color-subdued);
            }

            &:focus,
            &:focus-within {
                border-color: var(--theme--primary);
            }
        }

        &.edit-mode :deep(.interface>*),
        &.edit-mode :deep(.v-input .input),
        &.edit-mode :deep(.v-checkbox) {
            border-color: var(--theme--primary);
        }

        &:not(.edit-mode),
        & :deep(.v-input .input) {
            padding: calc(8px - var(--theme--border-width)) calc(12px - var(--theme--border-width));
        }

        & :deep(.v-input),
        & :deep(.v-checkbox) {
            --theme--form--field--input--height: 38px;
            min-height: 100%;
        }

        & :deep(.v-form),
        & :deep(.v-form > .field),
        & :deep(.v-form > .field > .interface),
        & :deep(.v-form .collection-item-dropdown),
        & :deep(.v-form .collection-item-dropdown .render-template),
        & :deep(.v-form .collection-item-dropdown .preview),
        & :deep(.v-form .many-to-one),
        & :deep(.v-form .many-to-one .preview),
        & :deep(.v-form .many-to-one .preview .render-template),
        &.input-autocomplete-api :deep(.interface > div) {
            height: 100%;
        }

        & :deep(.v-form) {
            z-index: 1;
            width: 100%;
        }

        &.edit-mode :deep(.v-form) {
            z-index: 3;
        }

        & :deep(.v-form > .field) {
            min-width: min-content;
        }

        &.collection-item-dropdown :deep(.v-form > .field) {
            min-width: 120px;
        }

        &.select-color {
            & :deep(.prepend) {
                line-height: 0;
            }

            & :deep(.prepend .v-button) {
                width: auto;
                min-width: 24px;
                max-height: none;
                aspect-ratio: 1;
                margin-left: 0;
            }

            & :deep(.prepend .v-button > button) {
                width: 100%;
                height: 100%;
            }
        }

        &.slider {
            & :deep(.field) {
                border: var(--theme--border-width) solid var(--theme--primary);
                border-radius: var(--theme--border-radius);
                padding-inline: calc(12px - var(--theme--border-width));
                box-shadow: var(--theme--form--field--input--box-shadow-focus);
                background-color: var(--v-input-background-color, var(--theme--form--field--input--background));
            }

            & :deep(.v-slider .slider input) {
                background-color: transparent;
            }

            & :deep(.v-slider) {
                position: relative;
                top: 50%;
                margin-top: -6px;
            }

            & :deep(.v-slider .slider.thumb-label-visible) {
                margin-bottom: 0;
            }
        }

        .v-slider .slider .thumb-label &:focus .edit-marker {
            display: none;
        }

        &.file {
            & :deep(.v-input) {
                width: 100%;
                min-width: max-content;
            }

            & :deep(.file),
            & :deep(.v-menu-activator > div) {
                height: 100%;
            }

            & :deep(.prepend .preview) {
                height: 32px;
                width: 32px;
                margin-left: -4px;
            }
        }

        .edit-marker {
            position: absolute;
            left: 1px;
            top: 50%;
            transform: translate(0, -50%);
            display: block;
            width: 5px;
            height: 5px;
            border-radius: 5px;
            background-color: var(--theme--primary);

        }
    }

    :global(.align-right .spreadsheet-cell .edit-marker) {
        left: auto !important;
        right: 1px;
    }
</style>