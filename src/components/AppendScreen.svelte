<script>
    import {createEventDispatcher} from 'svelte';
    import IcoEdit from "./icons/IcoEdit.svelte";
    import IcoList from "./icons/IcoList.svelte";

    const dispatch = createEventDispatcher();

    export let tabsUrls;

    let from = '';
    let to = '';
    let cookie = '';
    let toFieldIsSelect = true;

    let disabled = true;
    let fromCookies = [];

    let toInput;

    function changeForm() {
        return isFormValid() ? dispatch('addPair', {from, to, cookie}) : false;
    }

    function changeFromSelect(url) {
        from = url;
        chrome.cookies.getAll({url}, function (cookies) {
            fromCookies = cookies.map((cookie) => cookie.name);
            changeCookieSelect(fromCookies[0])
            setDisabled();
        });
    }

    function changeToSelect(url) {
        to = url;
        setDisabled();
    }

    function changeCookieSelect(name) {
        cookie = name ? name : false;
        setDisabled();
    }

    function setDisabled() {
        disabled = !isFormValid();
    }

    function isFormValid() {
        return (from && to && cookie);
    }

    function toggleToField() {
        toFieldIsSelect = !toFieldIsSelect;
        if (toFieldIsSelect) {
            to = '';
            setDisabled();
        } else {
            setTimeout(() => {
                toInput.focus()
            }, 0);
        }
    }
</script>
<form class="form" on:submit|preventDefault={changeForm}>
    <table class="table">
        <tr>
            <td>
                <label for="from" class="form__label">From</label>
            </td>
            <td colspan="2">
                <div class="form__select">
                    <select name="from" id="from" on:change={({target}) => {changeFromSelect(target.value)}}>
                        <option disabled selected>Select tab</option>
                        {#each tabsUrls as url, i}
                            <option value="{url}">{url}</option>
                        {/each}
                    </select>
                </div>
            </td>
        </tr>
        <tr>
            <td>
                {#if toFieldIsSelect}
                    <label for="toSelect" class="form__label">To</label>
                {:else}
                    <label for="toInput" class="form__label">To</label>
                {/if}
            </td>
            <td>
                {#if toFieldIsSelect}
                    <div class="form__select">
                        <select name="to" id="toSelect" on:change={({target}) => {changeToSelect(target.value)}}>
                            <option disabled selected>Select tab</option>
                            {#each tabsUrls as url, i}
                                <option value="{url}">{url}</option>
                            {/each}
                        </select>
                    </div>
                {:else}
                    <div class="form__input">
                        <input bind:this={toInput} type="text" id="toInput" bind:value={to} on:change={setDisabled}>
                    </div>
                {/if}
            </td>
            <td>
                <button type="button" class="button button--square button--bordered" on:click={toggleToField}>
                    {#if toFieldIsSelect}
                        <IcoEdit/>
                    {:else}
                        <IcoList/>
                    {/if}
                </button>
            </td>
        </tr>
        <tr>
            <td>
                <label for="cookie" class="form__label">Cookie</label>
            </td>
            <td colspan="2">
                <div class="form__select">
                    <select name="cookie" id="cookie"
                            on:change={({target}) => {changeCookieSelect(target.value)}}>
                        {#each fromCookies as cookie, i}
                            <option value="{cookie}">{cookie}</option>
                        {/each}
                    </select>
                </div>
            </td>
        </tr>
        <tr>
            <td colspan="3">
                <button type="submit" class="button button--green" {disabled}>Save</button>
            </td>
        </tr>
    </table>
</form>
