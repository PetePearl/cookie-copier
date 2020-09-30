<script>
    import AppendWindow from './components/AppendScreen.svelte';
    import PairsWindow from './components/PairsScreen.svelte';
    import Header from "./components/Header.svelte";
    import IcoSync from "./components/icons/IcoSync.svelte";
    import IcoPlus from "./components/icons/IcoPlus.svelte";
    import IcoBack from "./components/icons/IcoBack.svelte";
    import EditScreen from "./components/EditScreen.svelte";

    /** Pseudo Enum for each all screens in application */
    const screens = {
        pairs: 'pairs',
        append: 'append',
        edit: 'edit',
    }

    /** State */
    let state = {
        currentScreen: screens.pairs,
        screens: {
            pairs: {
                title: 'Cookie copier',
            },
            append: {
                title: 'Add a new pair'
            },
            edit: {
                title: 'Edit pair'
            }
        },

        /**
         * Pairs of added sites for cloning cookies
         * value will be taken from chrome.storage.sync
         *
         * from: string
         * to: string
         * cookie: string
         * isIdentical: boolean
         */
        pairs: [],
    }

    /** Pair for edit screen */
    let editablePair;

    /** Array of current open tabs */
    let tabsUrls = [];

    /** Show pairs screen */
    function setPairsScreen() {
        state.currentScreen = screens.pairs;
    }

    /** Show append screen */
    function setAppendScreen() {
        chrome.tabs.query({}, function (tabs) {
            tabsUrls = tabs.map((tab) => tab.url).filter(url => !!url)
            state.currentScreen = screens.append;
        });
    }

    /** Show edit screen */
    function setEditScreen({detail}) {
        editablePair = detail;
        state.currentScreen = screens.edit;
    }

    /** Set initial value of pairs from chrome storage sync */
    (function setPairsFromStorage() {
        chrome.storage.sync.get(['pairs'], function ({pairs}) {
            state.pairs = pairs || [];
            checkIsIdentical();
        });
    })()

    /** Check all pairs for cookies identical */
    function checkIsIdentical() {
        state.pairs.map((pair) => {
            chrome.cookies.get({url: pair.from, name: pair.cookie}, function (cookieFrom) {
                if (cookieFrom) {
                    let fromCookieValue = cookieFrom.value;

                    chrome.cookies.get({url: pair.to, name: pair.cookie}, function (cookieTo) {
                        let toCookieValue = cookieTo ? cookieTo.value : false;
                        pair.isIdentical = fromCookieValue === toCookieValue;

                        state.pairs = [...state.pairs];
                    })
                }
            });
        });
    }

    /** Save current pairs to chrome storage sync */
    function updateStorage() {
        chrome.storage.sync.set({
            pairs: state.pairs.map((pair) => {
                delete pair.isIdentical;
                return pair;
            })
        });
    }

    /** Add new pair to pairs list */
    function addNewPair({detail}) {
        state.pairs = [...state.pairs, detail];
        updateStorage();
        setPairsScreen();
        checkIsIdentical();

        cloneCookieForPair({detail});
    }

    /** Remove pair */
    function removePair({detail}) {
        state.pairs = state.pairs.filter(pair => pair !== detail)
        updateStorage();
        checkIsIdentical();
    }


    /** Edit pair */
    function swapPair({detail}) {
        state.pairs = state.pairs.map(pair => pair === editablePair ? detail : editablePair);
        editablePair = {};
        updateStorage();
        checkIsIdentical();
        setPairsScreen();
    }

    /** Clone cookie value for pair */
    function cloneCookieForPair({detail}) {
        chrome.cookies.get({url: detail.from, name: detail.cookie}, function (cookieFrom) {
            if (cookieFrom) {
                let fromCookieValue = cookieFrom.value;

                chrome.cookies.set({url: detail.to, name: detail.cookie, value: fromCookieValue}, function () {
                    detail.isIdentical = true;

                    checkIsIdentical();
                })
            }
        });
    }
</script>

<div class="app">
    <Header title="{state.screens[state.currentScreen].title}"/>

    <div class="app__body">
        {#if state.currentScreen === screens.pairs}

            {#if !state.pairs.length}
                <div class="first">
                    Append your first pair &nbsp;&nbsp;
                    <button class="button button--square button--green" on:click={setAppendScreen}>
                        <IcoPlus/>
                    </button>
                </div>
            {:else}
                <div class="app__button app__button--left">
                    <button class="button button--green" on:click={setAppendScreen}>
                        <IcoPlus/>
                    </button>
                </div>

                <div class="app__button app__button--right">
                    <button class="button" on:click={checkIsIdentical}>
                        <IcoSync/>
                    </button>
                </div>
            {/if}

            <PairsWindow
                    on:cloneCookieForPair={cloneCookieForPair}
                    on:removePair={removePair}
                    on:editPair={setEditScreen}
                    pairs="{state.pairs}"
            />

        {:else if state.currentScreen === screens.append}
            <div class="app__button app__button--left">
                <button class="button button--bordered" on:click={setPairsScreen}>
                    <IcoBack/>
                </button>
            </div>
            <AppendWindow on:addPair="{addNewPair}" tabsUrls="{tabsUrls}"/>

        {:else if state.currentScreen === screens.edit}
            <div class="app__button app__button--left">
                <button class="button button--bordered" on:click={setPairsScreen}>
                    <IcoBack/>
                </button>
            </div>
            {#if editablePair}
                <EditScreen on:swapPair={swapPair} fromUrl="{editablePair.from}" toUrl="{editablePair.to}"
                            cookie="{editablePair.cookie}"/>
            {/if}
        {/if}
    </div>
</div>
