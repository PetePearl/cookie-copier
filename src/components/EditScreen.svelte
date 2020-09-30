<script>
    import {createEventDispatcher} from 'svelte';

    const dispatch = createEventDispatcher();

    function swapPair() {
        dispatch('swapPair', {from: fromUrl, to: toUrl, cookie})
    }

    export let fromUrl;
    export let toUrl;
    export let cookie;

    let disabled = false;

    function setDisabled() {
        disabled = !isFormValid();
    }

    function isFormValid() {
        return (fromUrl && toUrl && cookie);
    }
</script>

<form class="form" on:submit|preventDefault={swapPair}>
    <table class="table">
        <tr>
            <td>
                <label for="from" class="form__label">From</label>
            </td>
            <td>
                <div class="form__input">
                    <input type="text" id="from" bind:value={fromUrl} on:change={setDisabled}>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <label for="to" class="form__label">To</label>
            </td>
            <td>
                <div class="form__input">
                    <input type="text" id="to" bind:value={toUrl} on:change={setDisabled}>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                <label for="cookie" class="form__label">Cookie</label>
            </td>
            <td>
                <div class="form__input">
                    <input type="text" id="cookie" bind:value={cookie} on:change={setDisabled}>
                </div>
            </td>
        </tr>
        <tr>
            <td colspan="2">
                <button type="submit" class="button button--green" {disabled}>Save</button>
            </td>
        </tr>
    </table>
</form>

