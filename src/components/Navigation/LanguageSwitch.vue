<template>
  <div class="navigation__language-switch">
    <div v-tooltip="$t('language-title')" class="navigation__button" @click="toggleLanguageMenu">
      <FontAwesomeIcon class="navigation__language-icon" icon="language" fixedWidth></FontAwesomeIcon>
    </div>

    <transition name="navigation__language-picker">
      <div v-if="languageMenu" class="navigation__language-picker">
        <div v-for="locale in $i18n.availableLocales" :key="locale" class="navigation__language" :class="{ 'navigation__language--active': $i18n.locale === locale }" @click.prevent="changeLocale(locale)">
          <Flag v-tooltip="locale" :country="getFlagCountry(locale)"></Flag>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
  import { mapActions, mapGetters } from 'vuex';
  import * as storage from '../../utils/storage';
  import isAprilFoolsDay from '../../utils/isAprilFoolsDay';
  import Flag from '../utils/Flag.vue';

  export default {
    name: 'NavigationLanguageSwitch',
    components: { Flag },
    computed: {
      ...mapGetters({
        languageMenu: 'layout/languageMenu',
      }),
    },
    watch: {
      languageMenu(value) {
        if (value) window.addEventListener('click', this.onWindowClick);
        else window.removeEventListener('click', this.onWindowClick);
      },
    },
    beforeDestroy() {
      window.removeEventListener('click', this.onWindowClick);
    },
    methods: {
      ...mapActions({
        toggleLanguageMenu: 'layout/toggleLanguageMenu',
      }),
      getFlagCountry(locale) {
        if (locale === 'sr-CS') return 'rs';
        if (locale === 'lol-US') return 'lol';
        return locale.split('-')[1].toLowerCase();
      },
      displayTranslationStatus() {
        const { translationPercent } = this.$i18n;

        if (translationPercent === 100) return; // Nothing to do here
        if (translationPercent > 80) return this.$info(this.$t('language-translation-good', { percent: translationPercent.toFixed(2), locale: this.$i18n.locale }));
        if (translationPercent > 40) return this.$info(this.$t('language-translation-medium', { percent: translationPercent.toFixed(2), locale: this.$i18n.locale }));
        return this.$info(this.$t('language-translation-bad', { percent: translationPercent.toFixed(2), locale: this.$i18n.locale }));
      },
      async changeLocale(locale) {
        const year = new Date().getFullYear();
        if (isAprilFoolsDay()) storage.set(`fooled-${year}`, true);

        await this.$i18n.load(locale);
        await this.$i18n.set(locale);
        storage.set('locale', locale);

        this.displayTranslationStatus();

        this.$store.dispatch('layout/toggleLanguageMenu');
      },
      onWindowClick($e) {
        const path = $e.path || $e.composedPath();
        if (path.includes(this.$el)) return;
        this.toggleLanguageMenu();
      },
    },
  };
</script>

<style lang="scss">
	.navigation__language-switch {
		display: flex;
		justify-content: center;
		position: relative;
	}

	.navigation__language-icon {
		pointer-events: none;
	}

	.navigation__language-picker {
		background: var(--color-theme);
		display: grid;
		grid-gap: 0.25em;
		grid-template-columns: repeat(6, auto);
		padding: 0.5em;
		position: absolute;
		right: 0;
		top: var(--navigation-height);
		transform-origin: top;
		transition: transform .3s;

		@media screen and (max-width: 600px) {
			grid-template-columns: repeat(5, auto);
		}
	}

	.navigation__language {
		border-radius: 4px;
		cursor: pointer;
		padding: 0.5em;

		&:hover {
			background: var(--color-theme-dark);
		}
	}

	.navigation__language--active {
		background: var(--color-theme-dark);
		box-shadow: inset 0 0 1px 0 var(--color-navigation);
	}

	.navigation__language-picker-enter, .navigation__language-picker-leave-to {
		transform: scaleY(0);
	}
</style>
