<script>
    import {createEventDispatcher} from 'svelte';

    const dispatch = createEventDispatcher();

    import IcoCopy from "./icons/IcoCopy.svelte";
    import IcoDelete from "./icons/IcoDelete.svelte";
    import IcoEdit from "./icons/IcoEdit.svelte";

    export let pairs;

    function removePair(pair) {
        dispatch('removePair', pair);
    }

    function cloneCookieForPair(pair) {
        dispatch('cloneCookieForPair', pair);
    }

    function editPair(pair) {
        dispatch('editPair', pair);
    }
</script>
<div class="pairs">
    {#each pairs as pair, i}
        <div class="pair" class:pair--identical={pair.isIdentical}>
            <button class="pair__copy" on:click={() => {cloneCookieForPair(pair)}}>
                <IcoCopy/>
            </button>
            <div class="pair__points">
                <div class="pair__point">
                    <b>From url:</b>
                    {pair.from}
                </div>
                <div class="pair__point">
                    <b>To url:</b>
                    {pair.to}
                </div>
                <div class="pair__point">
                    <b>Cookie:</b>
                    {pair.cookie}
                </div>
            </div>
            <div class="pair__controls">
                <button on:click={() => {editPair(pair)}} class="pair__control pair__control--edit">
                    <IcoEdit/>
                </button>
                <button on:click={() => {removePair(pair)}} class="pair__control pair__control--remove">
                    <IcoDelete/>
                </button>
            </div>
        </div>
    {/each}
</div>
