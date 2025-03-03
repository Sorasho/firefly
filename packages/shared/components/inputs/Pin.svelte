<script lang="typescript">
    import { onMount } from 'svelte'
    import { createEventDispatcher } from 'svelte'
    import { validatePinFormat, PIN_LENGTH } from 'shared/lib/utils'
    import { Error, Icon } from 'shared/components'

    const dispatch = createEventDispatcher()

    export let value = undefined
    export let classes = ''
    export let disabled = false
    export let autofocus = false
    export let glimpse = false
    export let smaller = false
    export let error

    let inputs = new Array(PIN_LENGTH)
    $: {
        if (!value) {
            inputs = new Array(PIN_LENGTH)
        }
    }

    let root
    const inputElements = []

    const KEYBOARD = {
        BACKSPACE: 8,
        ENTER: 13,
        TAB: 9,
    }

    onMount(() => {
        if (autofocus) {
            focus()
        }
    })

    const handleBackspace = () => {
        // Search for the last child with a value
        // and remove it
        for (let j = 1; j <= PIN_LENGTH; j++) {
            if (j === PIN_LENGTH || !inputs[j]) {
                inputs[j - 1] = ''
                inputElements[j - 1].focus()
                break
            }
        }
        value = inputs.join('')
    }

    const changeHandler = function (e, i) {
        const regex = new RegExp(/^\d+$/)

        if (e.keyCode == KEYBOARD.BACKSPACE) {
            handleBackspace()
        } else if (e.keyCode == KEYBOARD.ENTER) {
            if (validatePinFormat(inputs.join(''))) {
                dispatch('submit')
            }
        } else if (e.keyCode == KEYBOARD.TAB) {
            // Do default tab handling by focusing the root
            // container and allow default processing to happen
            root.focus()
            return
        } else {
            if (regex.test(e.key)) {
                // Search from the first child to find the first
                // empty value and start filling from there
                for (let j = 0; j < PIN_LENGTH; j++) {
                    if (!inputs[j]) {
                        inputs[j] = e.key
                        if (j < PIN_LENGTH - 1) {
                            inputElements[j + 1].focus()
                        }
                        break
                    }
                }
                value = inputs.join('')
            }
        }
        e.preventDefault()
    }

    const selectFirstEmpty = () => {
        for (let j = 0; j < PIN_LENGTH; j++) {
            if (!inputs[j] || j === PIN_LENGTH - 1) {
                inputElements[j].focus()
                return
            }
        }
    }

    const selectFirstEmptyRoot = (e): void => {
        if (e.target === root) {
            selectFirstEmpty()
        }
    }

    export function focus(): void {
        if (!disabled) {
            selectFirstEmpty()
        }
    }

    export function resetAndFocus(): void {
        if (!disabled) {
            inputs = new Array(PIN_LENGTH)
            selectFirstEmpty()
        } else {
            setTimeout(() => resetAndFocus(), 100)
        }
    }
</script>

<style type="text/scss">
    pin-input {
        @apply cursor-pointer;
        @apply select-none;

        &:not(.disabled):focus-within,
        &:not(.disabled):hover {
            @apply border-gray-500;
        }
        &.disabled {
            @apply pointer-events-none;
            @apply opacity-50;
        }
        .inputs-wrapper {
            height: 27px;
        }
        .input-wrapper {
            max-width: 204px;
            height: 27px;

            input {
                -webkit-text-security: none;
                @apply w-4;
                @apply bg-transparent;
                @apply opacity-0;
                @apply text-transparent;
                @apply cursor-pointer;
                @apply text-center;
                @apply text-18;
                caret-color: transparent;
                transition: opacity 1s, color 1s;

                &.active {
                    border-bottom-width: 1px;
                    border-bottom-style: solid;
                    @apply border-blue-500;
                    @apply cursor-text;
                    @apply opacity-100;

                    &.glimpse {
                        @apply text-blue-500;
                    }
                }

                &:focus {
                    @apply outline-none;
                }
            }
        }
        .input-decorator-wrapper {
            z-index: -1;
            max-width: 204px;
            height: 27px;
            input-decorator {
                @apply w-4;
                @apply h-4;
                @apply rounded-full;
                @apply bg-blue-500;
                @apply opacity-0;
                &.active {
                    transition: opacity 1s;
                    @apply opacity-100;
                }
            }
        }
    }
</style>

<div class="w-full {classes}">
    <pin-input
        style="--pin-input-size: {PIN_LENGTH}"
        class={`flex items-center justify-between w-full relative z-0 rounded-xl border border-solid
            bg-gray-50 dark:bg-gray-800 border-gray-300 dark:border-gray-700
            ${smaller ? 'h-14 pl-6 pr-4' : 'h-20 pl-12 pr-8'}`}
        class:disabled
        bind:this={root}
        on:click={selectFirstEmptyRoot}
        on:focus={selectFirstEmptyRoot}
        tabindex="0">
        <div class="flex flex-row inputs-wrapper">
            <div class="input-wrapper absolute items-center w-full flex flex-row flex-no-wrap justify-between">
                {#each inputs as item, i}
                    <input
                        bind:value={inputs[i]}
                        maxLength="1"
                        id={`input-${i}`}
                        type="text"
                        bind:this={inputElements[i]}
                        class:active={!inputs[i] || inputs[i].length === 0}
                        class:glimpse
                        {disabled}
                        on:keydown={(event) => changeHandler(event, i)}
                        on:contextmenu|preventDefault />
                {/each}
            </div>
            <div class="input-decorator-wrapper items-center absolute w-full flex flex-row flex-no-wrap justify-between">
                {#each inputs as item, i}
                    <input-decorator class:active={inputs[i] && inputs[i].length !== 0} class:disabled />
                {/each}
            </div>
        </div>
        <button type="button" on:click={handleBackspace} {disabled} tabindex="-1">
            <Icon icon="backspace" classes={smaller ? 'text-blue-500' : 'text-gray-500'} />
        </button>
    </pin-input>
    {#if error}
        <Error {error} />
    {/if}
</div>
