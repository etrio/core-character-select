<template>
    <div class="container">
        <Modal v-if="deleteDialog">
            <Frame minWidth="30vw" maxWidth="30vw">
                <template v-slot:toolbar>
                    <Toolbar :hideExit="true"><span class="red--text">{{ locales.LABEL_DELETE }} {{
                            characters[characterIndex].name
                    }}?</span></Toolbar>
                </template>
                <template v-slot:content>
                    <div class="overline mb-8 mt-8 center">{{ locales.LABEL_CONFIRM_DELETE }}?</div>
                    <div class="overline mb-8 mt-8 center amber--text">
                        {{ characters[characterIndex].name }}
                    </div>
                    <div class="split split-full">
                        <Button class="mt-2 fill-half-width" color="red" @click="hideDeleteInterface">
                            {{ locales.LABEL_NO }}
                        </Button>
                        <Button class="ml-4 mt-2 fill-half-width" color="green" @click="deleteCharacter">
                            {{ locales.LABEL_YES }}
                        </Button>
                    </div>
                </template>
            </Frame>
        </Modal>
        <div class="stats pl-2 pr-2 pt-2 pb-2">
            <div class="stat-wrapper stack pl-2 pr-2 pb-6 pt-6" style="width: 100%; box-sizing: border-box !important">
                <!-- Name -->
                <div class="stat split split-full split-center">
                    <span class="stat-name overline grey--text text--lighten-1 pl-1 pr-2">imie: </span>
                    <span class="stat-text overline grey--text text--lighten-1 pl-6 pr-2">{{ getName }}</span>
                </div>
                <!-- Stats -->
                <div class="stat split split-full split-center pt-4">
                    <span class="stat-name overline grey--text text--lighten-1 pl-1 pr-2">czas: </span>
                    <span class="stat-text overline grey--text text--lighten-1 pl-6 pr-2"> {{ getHours }} H </span>
                </div>
                <div class="stat split split-full split-center pt-4">
                    <span class="stat-name overline grey--text text--lighten-1 pl-1 pr-2">gotówka: </span>
                    <span class="stat-text overline grey--text text--lighten-1 pl-6 pr-2"> ${{ getCash }} </span>
                </div>
                <div class="stat split split-full split-center pt-4">
                    <span class="stat-name overline grey--text text--lighten-1 pl-1 pr-2">bank: </span>
                    <span class="stat-text overline grey--text text--lighten-1 pl-6 pr-2"> ${{ getBank }} </span>
                </div>
            </div>
        </div>


        <!-- SELECT -->
        <div class="stats2 pl-2 pr-2 pt-2 pb-2">
            <div class="stat2-wrapper stack" style="width: 100%; box-sizing: border-box !important">
                <div class="buttons split split-center space-between" style="width: 100%; box-sizing: border-box">
                    <Button class="buttons2" color="red" @click="showDeleteInterface">
                        <p class="btn-text">USUŃ</p>
                    </Button>
                    <Button class="buttons2" color="blue" @click="selectCharacter">
                        <p class="btn-text">WYBIERZ</p>
                    </Button>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, defineAsyncComponent } from 'vue';
import { EXAMPLE_CHARACTER } from './utility/exampleCharacter';
import { CHARACTER_SELECT_WEBVIEW_EVENTS } from '../shared/events';
import { CHARACTER_SELECT_LOCALE } from '../shared/locale';

import { CHARACTER_SELECT_CONFIG } from '../shared/config';

const ComponentName = 'CharacterSelect';
export default defineComponent({
    name: ComponentName,
    components: {
        Button: defineAsyncComponent(() => import('@components/Button.vue')),
        Icon: defineAsyncComponent(() => import('@components/Icon.vue')),
        Modal: defineAsyncComponent(() => import('@components/Modal.vue')),
        Toolbar: defineAsyncComponent(() => import('@components/Toolbar.vue')),
        Frame: defineAsyncComponent(() => import('@components/Frame.vue')),
    },
    data() {
        return {
            MAX_CHARACTERS: 0,
            characterIndex: 0,
            characters: [],
            statNames: [],
            deleteDialog: false,
            locales: CHARACTER_SELECT_LOCALE,
        };
    },
    computed: {
        getTotalCharacters() {
            return this.characters.length;
        },
        getName() {
            if (!this.characters[this.characterIndex]) {
                return '';
            }

            return this.characters[this.characterIndex].name.replace(/_/g, ' ');
        },
        getHours() {
            if (!this.characters[this.characterIndex]) {
                return '';
            }

            if (!this.characters[this.characterIndex].hours) {
                return '';
            }

            return this.characters[this.characterIndex].hours.toFixed(2);
        },
        getBank() {
            if (!this.characters[this.characterIndex]) {
                return '';
            }

            if (!this.characters[this.characterIndex].bank) {
                return '';
            }

            return this.characters[this.characterIndex].bank.toFixed(2);
        },
        getCash() {
            if (!this.characters[this.characterIndex]) {
                return '';
            }

            if (!this.characters[this.characterIndex].cash) {
                return '';
            }

            return this.characters[this.characterIndex].cash.toFixed(2);
        },
    },
    methods: {
        setCharacters(characters) {
            this.characterIndex = 0;
            this.characters = characters;
            this.updateAppearance();
        },
        incrementIndex() {
            this.characterIndex += 1;
            if (this.characterIndex > this.characters.length - 1) {
                this.characterIndex = 0;
            }
            this.updateAppearance();

            if (!('alt' in window)) {
                return;
            }

            alt.emit('play:Sound', 'NAV_LEFT_RIGHT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
        },
        decrementIndex() {
            this.characterIndex -= 1;
            if (this.characterIndex <= -1) {
                this.characterIndex = this.characters.length - 1;
            }
            this.updateAppearance();

            if (!('alt' in window)) {
                return;
            }

            alt.emit('play:Sound', 'NAV_LEFT_RIGHT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
        },
        updateAppearance() {
            if (!('alt' in window)) {
                return;
            }

            alt.emit(CHARACTER_SELECT_WEBVIEW_EVENTS.UPDATE, this.characterIndex);
        },
        selectCharacter() {
            if (!('alt' in window)) {
                return;
            }

            alt.emit('play:Sound', 'SELECT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
            alt.emit(CHARACTER_SELECT_WEBVIEW_EVENTS.SELECT, this.characters[this.characterIndex]._id);
        },
        newCharacter() {
            if (!('alt' in window)) {
                return;
            }

            alt.emit(CHARACTER_SELECT_WEBVIEW_EVENTS.NEW);
            alt.emit('play:Sound', 'SELECT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
        },
        showDeleteInterface() {
            this.deleteDialog = true;

            if (!('alt' in window)) {
                return;
            }

            alt.emit('play:Sound', 'SELECT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
        },
        hideDeleteInterface() {
            this.deleteDialog = false;

            if (!('alt' in window)) {
                return;
            }

            alt.emit('play:Sound', 'SELECT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
        },
        deleteCharacter() {
            this.deleteDialog = false;

            if (!('alt' in window)) {
                return;
            }

            alt.emit(CHARACTER_SELECT_WEBVIEW_EVENTS.DELETE, this.characters[this.characterIndex]._id);
            alt.emit('play:Sound', 'SELECT', 'HUD_FRONTEND_DEFAULT_SOUNDSET');
        },
        setLocales(localeObject) {
            this.locales = localeObject;
            this.updateLocales(true);
        },
        updateLocales(skipDefault = false) {
            this.statNames = [
                /* Name of the stat, variable name inside character object */
                { varName: this.locales.LABEL_NAME, varRef: 'name' },
                { varName: this.locales.LABEL_AGE, varRef: 'age', useInfo: true },
                { varName: this.locales.LABEL_GENDER, varRef: 'gender', useInfo: true },
                { varName: this.locales.LABEL_HOURS, varRef: 'hours' },
                { varName: this.locales.LABEL_CASH, varRef: 'cash', prefix: '$' },
                { varName: this.locales.LABEL_BANK, varRef: 'bank', prefix: '$' },
            ];
        },
    },
    mounted() {
        if ('alt' in window) {
            this.MAX_CHARACTERS = CHARACTER_SELECT_CONFIG.MAX_CHARACTERS;
            alt.on(CHARACTER_SELECT_WEBVIEW_EVENTS.SET_CHARACTERS, this.setCharacters);
            alt.emit(`${ComponentName}:Ready`);
        } else {
            this.characters = [
                {
                    ...EXAMPLE_CHARACTER,
                    ...{ appearance: { sex: 0 }, name: 'Elon_Musk', bank: 25000, cash: 25, hours: 100 },
                },
                {
                    ...EXAMPLE_CHARACTER,
                    ...{ appearance: { sex: 0 }, name: 'Is_A', bank: 26000, cash: 35, hours: 255 },
                },
                {
                    ...EXAMPLE_CHARACTER,
                    ...{ appearance: { sex: 0 }, name: 'Fucking_Lizard', bank: 21111, cash: 409, hours: 69 },
                },
            ];
        }

        this.updateLocales();
    },
    unmounted() {
        if ('alt' in window) {
            alt.off(CHARACTER_SELECT_WEBVIEW_EVENTS.SET_CHARACTERS, this.setCharacters);
        }
    },
});
</script>

<style scoped>
.container {
    display: block;
    position: absolute;
    /* background: black; */
    width: 100%;
    height: 100%;
    background: linear-gradient(to top left, rgba(0, 0, 0, 1), transparent 35%);
}

.navigation {
    position: absolute;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    min-width: 40vw;
    max-width: 40vw;
    box-sizing: border-box;
    text-shadow: 0px 0px 1px black, 0px 0px 4px black;
    bottom: 0px;
    margin: 0 auto;
}

.nav-container {
    display: flex;
    justify-content: center;
    align-content: center;
    align-items: center;
    justify-items: center;
    min-width: 100vw;
    max-width: 100vw;
}

.stats {
    position: fixed;
    display: flex !important;
    flex-direction: column;
    justify-items: flex-start !important;
    align-items: flex-start !important;
    justify-content: flex-start !important;
    align-content: flex-start !important;
    background: rgba(11, 11, 11, 1);
    box-sizing: border-box;
    min-width: 350px;
    max-width: 350px;
    border: 2px solid rgba(32, 32, 32, 1);
    border-radius: 5px;
    overflow: hidden;
    top: 20px;
    right: 1vw;
}

.buttons2 {
    width: 100px;
    height: 60px;
    border: none;
    outline: none;
    background: rgb(12, 12, 12);
}


.stats2 {
    position: fixed;
    display: flex !important;
    flex-direction: column;
    justify-items: flex-start !important;
    align-items: flex-start !important;
    justify-content: flex-start !important;
    align-content: flex-start !important;
    background: rgb(12, 12, 12);
    box-sizing: border-box;
    min-width: 350px;
    max-width: 350px;
    min-height: 50px;
    border: 2px solid rgb(12, 12, 12);
    border-radius: 10px;
    overflow: hidden;
    bottom: 50px;
    left: 36%;
}

.stat2-wrapper {
    border: 2px solid rgb(32, 32, 32);
}

.stat2 {
    box-sizing: border-box;
}

.stat {
    box-sizing: border-box;
}

.stat-text {
    width: 100%;
    text-align: right;
    font-size: 14px !important;
}

.stat-icon {
    min-width: 30px;
    max-width: 30px;
}

.stat-name {
    min-width: 30px;
    max-width: 30px;
    font-size: 14px !important;
}


.stat-wrapper {
    border: 2px solid rgb(70, 70, 70);
    border-radius: 10px;
}
</style>
